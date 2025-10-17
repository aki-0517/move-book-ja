---
title: '参照 | リファレンス'
description: ''
---

# 参照

Moveには2つの型の参照があります：不変`&`と可変`&mut`。不変参照は読み取り専用であり、基になる値（またはそのフィールド）を変更することはできません。可変参照は、その参照を通じた書き込みによる変更を可能にします。Moveの型システムは、参照エラーを防ぐ所有権の規律を強制します。

## 参照演算子

Moveは、参照の作成と拡張、および可変参照を不変参照に変換するための演算子を提供します。ここおよび他の場所では、「式`e`が型`T`を持つ」ことを表すために`e: T`という記法を使用します。

| 構文      | 型                                                  | 説明                                                    |
| ----------- | ----------------------------------------------------- | -------------------------------------------------------------- |
| `&e`        | `&T`（`e: T`かつ`T`が非参照型）     | `e`への不変参照を作成                           |
| `&mut e`    | `&mut T`（`e: T`かつ`T`が非参照型） | `e`への可変参照を作成                             |
| `&e.f`      | `&T`（`e.f: T`）                                   | 構造体`e`のフィールド`f`への不変参照を作成      |
| `&mut e.f`  | `&mut T`（`e.f: T`）                               | 構造体`e`のフィールド`f`への可変参照を作成          |
| `freeze(e)` | `&T`（`e: &mut T`）                                | 可変参照`e`を不変参照に変換 |

`&e.f`と`&mut e.f`演算子は、構造体への新しい参照を作成するためにも、既存の参照を拡張するためにも使用できます：

```move
let s = S { f: 10 };
let f_ref1: &u64 = &s.f; // 動作する
let s_ref: &S = &s;
let f_ref2: &u64 = &s_ref.f // これも動作する
```

複数のフィールドを持つ参照式は、両方の構造体が同じモジュールにある限り動作します：

```move
public struct A { b: B }
public struct B { c : u64 }
fun f(a: &A): &u64 {
    &a.b.c
}
```

最後に、参照への参照は許可されていないことに注意してください：

```move
let x = 7;
let y: &u64 = &x;
// highlight-error
let z: &&u64 = &y; // エラー！コンパイルされません
```

## 参照を通じた読み取りと書き込み

可変参照と不変参照の両方で、参照された値のコピーを生成するために読み取りができます。

可変参照のみが書き込み可能です。書き込み`*x = v`は、`x`に以前に格納されていた値を破棄し、`v`で更新します。

両方の操作はC言語風の`*`構文を使用します。ただし、読み取りは式であるのに対し、書き込みは等号の左側で発生しなければならない変更であることに注意してください。

| 構文     | 型                                | 説明                         |
| ---------- | ----------------------------------- | ----------------------------------- |
| `*e`       | `T`（`e`が`&T`または`&mut T`）   | `e`が指す値を読み取り    |
| `*e1 = e2` | `()`（`e1: &mut T`かつ`e2: T`） | `e1`の値を`e2`で更新 |

参照を読み取るためには、基になる型が[`copy`アビリティ](../abilities)を持っている必要があります。これは、参照を読み取ると値の新しいコピーが作成されるためです。このルールは、アセットのコピーを防ぎます：

```move
fun copy_coin_via_ref_bad(c: Coin) {
    let c_ref = &c;
    // highlight-error
    let counterfeit: Coin = *c_ref; // 許可されません！
    pay(c);
    pay(counterfeit);
}
```

双対的に：参照に書き込むためには、基になる型が[`drop`アビリティ](../abilities)を持っている必要があります。これは、参照に書き込むと古い値が破棄（または「ドロップ」）されるためです。このルールは、リソース値の破壊を防ぎます：

```move
fun destroy_coin_via_ref_bad(mut ten_coins: Coin, c: Coin) {
    let ref = &mut ten_coins;
    // highlight-error
    *ref = c; // エラー！許可されません--10コインが破壊されます！
}
```

## `freeze`推論

可変参照は、不変参照が期待されるコンテキストで使用できます：

```move
let mut x = 7;
let y: &u64 = &mut x;
```

これは、内部でコンパイラが必要な場所に`freeze`命令を挿入するためです。以下は、`freeze`推論の動作例です：

```move
fun takes_immut_returns_immut(x: &u64): &u64 { x }

// 戻り値でのfreeze推論
fun takes_mut_returns_immut(x: &mut u64): &u64 { x }

fun expression_examples() {
    let mut x = 0;
    let mut y = 0;
    takes_immut_returns_immut(&x); // 推論なし
    takes_immut_returns_immut(&mut x); // 推論されたfreeze(&mut x)
    takes_mut_returns_immut(&mut x); // 推論なし

    assert!(&x == &mut y, 42); // 推論されたfreeze(&mut y)
}

fun assignment_examples() {
    let x = 0;
    let y = 0;
    let imm_ref: &u64 = &x;

    imm_ref = &x; // 推論なし
    imm_ref = &mut y; // 推論されたfreeze(&mut y)
}
```

### サブタイピング

この`freeze`推論により、Move型チェッカーは`&mut T`を`&T`のサブタイプとして見ることができます。上記で示したように、これは`&T`値が使用される任意の式の任意の場所で、`&mut T`値も使用できることを意味します。この用語は、`&T`が提供された場所で`&mut T`が必要だったことを簡潔に示すためにエラーメッセージで使用されます。例えば

```move
module a::example {
    fun read_and_assign(store: &mut u64, new_value: &u64) {
        *store = *new_value
    }

    fun subtype_examples() {
        let mut x: &u64 = &0;
        let mut y: &mut u64 = &mut 1;

        x = &mut 1; // valid
        // highlight-error
        y = &2; // ERROR! invalid!

        read_and_assign(y, x); // valid
        // highlight-error
        read_and_assign(x, y); // ERROR! invalid!
    }
}
```

will yield the following error messages

```text
error:

    ┌── example.move:11:9 ───
    │
 12 │         y = &2; // invalid!
    │         ^ Invalid assignment to local 'y'
    ·
 12 │         y = &2; // invalid!
    │             -- The type: '&{integer}'
    ·
  9 │         let mut y: &mut u64 = &mut 1;
    │                    -------- Is not a subtype of: '&mut u64'
    │

error:

    ┌── example.move:14:9 ───
    │
 15 │         read_and_assign(x, y); // invalid!
    │         ^^^^^^^^^^^^^^^^^^^^^ Invalid call of 'a::example::read_and_assign'. Invalid argument for parameter 'store'
    ·
  8 │         let mut x: &u64 = &0;
    │                    ---- The type: '&u64'
    ·
  3 │     fun read_and_assign(store: &mut u64, new_value: &u64) {
    │                                -------- Is not a subtype of: '&mut u64'
    │
```

現在サブタイピングを持つ他の型は[タプル](./tuples)のみです

## 所有権

可変参照と不変参照の両方は、_同じ参照の既存のコピーや拡張がある場合でも_、常にコピーおよび拡張できます：

```move
fun reference_copies(s: &mut S) {
  let s_copy1 = s; // ok
  let s_extension = &mut s.f; // これもok
  let s_copy2 = s; // まだok
  ...
}
```

これは、Rustの所有権システムに慣れているプログラマーにとって驚くべきことかもしれません。Rustは上記のコードを拒否するでしょう。Moveの型システムは[コピー](./../variables#move-and-copy)の処理においてより寛容ですが、書き込み前の可変参照の一意の所有権を確保する点では同様に厳格です。

### 参照は格納できない

参照とタプルは、構造体のフィールド値として格納できない_唯一の_型です。これは、それらがストレージや[オブジェクト](./../abilities/object)に存在できないことも意味します。プログラム実行中に作成されたすべての参照は、Moveプログラムが終了するときに破棄されます。それらは完全に一時的です。これは`store`アビリティを持たないすべての型にも適用されます：非`store`型の任意の値は、プログラムが終了する前に破棄されなければなりません。

これは、構造体内部に参照を格納することを許可するRustとのもう1つの違いです。

構造体に参照を格納することを許可する、より洗練された、より表現力豊かな型システムを想像することができます。`store`[アビリティ](./../abilities)を持たない構造体内部の参照を許可することもできますが、核心的な困難は、Moveが静的参照安全性を追跡するためのかなり複雑なシステムを持っていることです。型システムのこの側面も、構造体内部に参照を格納することをサポートするために拡張する必要があります。要するに、Moveの参照安全性システムは、格納された参照をサポートするために拡張する必要があり、言語が進化するにつれて注目しているものです。

<!-- TODO 借用ルールのスケッチを実際に文書化する -->
