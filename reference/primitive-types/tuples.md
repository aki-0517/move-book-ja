---
title: 'タプルとユニット | リファレンス'
description: ''
---

# タプルとユニット

Moveは、[第一級値](https://en.wikipedia.org/wiki/First-class_citizen)としてタプルを持つ他の言語から期待されるように、タプルを完全にサポートしていません。ただし、複数の戻り値をサポートするために、Moveにはタプル風の式があります。これらの式は実行時に具体的な値を生成せず（バイトコードにタプルは存在しません）、その結果非常に制限されています：

- それらは式でのみ現れる（通常は関数の戻り位置）。
- それらはローカル変数にバインドできない。
- それらは構造体に格納できない。
- タプル型はジェネリクスのインスタンス化に使用できない。

同様に、[ユニット`()`](https://en.wikipedia.org/wiki/Unit_type)は、式ベースにするためにMoveソース言語によって作成された型です。ユニット値`()`は、ランタイム値は生成しません。ユニット`()`を空のタプルと考えることができ、タプルに適用される任意の制限はユニットにも適用されます。

これらの制限を考えると、言語にタプルがあることが奇妙に感じるかもしれません。しかし、他の言語でのタプルの最も一般的な使用例の1つは、関数が複数の値を返すことを可能にすることです。一部の言語は、複数の戻り値を含む構造体をユーザーに書かせることでこれを回避しています。しかし、Moveでは[構造体](./../structs)内部に参照を置くことができません。これにより、Moveは複数の戻り値をサポートする必要がありました。これらの複数の戻り値はすべて、バイトコードレベルでスタックにプッシュされます。ソースレベルでは、これらの複数の戻り値はタプルを使用して表現されます。

## リテラル

タプルは、括弧内のカンマ区切りの式リストによって作成されます。

| 構文          | 型                                                                         | 説明                                                  |
| --------------- | ---------------------------------------------------------------------------- | ------------------------------------------------------------ |
| `()`            | `(): ()`                                                                     | ユニット、空のタプル、またはアリティ0のタプル               |
| `(e1, ..., en)` | `(e1, ..., en): (T1, ..., Tn)`（`e_i: Ti`で`0 < i <= n`かつ`n > 0`） | `n`タプル、アリティ`n`のタプル、`n`個の要素を持つタプル |

`(e)`は型`(e): (t)`を持たないことに注意してください。言い換えれば、1つの要素を持つタプルはありません。括弧内に単一の要素しかない場合、括弧は曖昧性の解決にのみ使用され、他の特別な意味は持ちません。

時々、2つの要素を持つタプルは「ペア」と呼ばれ、3つの要素を持つタプルは「トリプル」と呼ばれます。

### 例

```move
module 0::example;

// これら3つの関数はすべて同等

// 戻り型が提供されない場合、`()`と仮定される
fun returns_unit_1() { }

// 空の式ブロックには暗黙の()値がある
fun returns_unit_2(): () { }

// `returns_unit_1`と`returns_unit_2`の明示的バージョン
fun returns_unit_3(): () { () }


fun returns_3_values(): (u64, bool, address) {
    (0, false, @0x42)
}
fun returns_4_values(x: &u64): (&u64, u8, u128, vector<u8>) {
    (x, 0, 1, b"foobar")
}
```

## 操作

現在、タプルに対して実行できる唯一の操作は分割代入です。

### 分割代入

任意のサイズのタプルは、`let`バインディングまたは代入で分割代入できます。

例：

```move
module 0x42::example;

// all 3 of these functions are equivalent
fun returns_unit() {}
fun returns_2_values(): (bool, bool) { (true, false) }
fun returns_4_values(x: &u64): (&u64, u8, u128, vector<u8>) { (x, 0, 1, b"foobar") }

fun examples(cond: bool) {
    let () = ();
    let (mut x, mut y): (u8, u64) = (0, 1);
    let (mut a, mut b, mut c, mut d) = (@0x0, 0, false, b"");

    () = ();
    (x, y) = if (cond) (1, 2) else (3, 4);
    (a, b, c, d) = (@0x1, 1, true, b"1");
}

fun examples_with_function_calls() {
    let () = returns_unit();
    let (mut x, mut y): (bool, bool) = returns_2_values();
    let (mut a, mut b, mut c, mut d) = returns_4_values(&0);

    () = returns_unit();
    (x, y) = returns_2_values();
    (a, b, c, d) = returns_4_values(&1);
}
```

詳細については、[Move変数](./../variables)を参照してください。

## サブタイピング

参照と同様に、タプルはMoveで[サブタイピング](https://en.wikipedia.org/wiki/Subtyping)を持つ唯一の型です。タプルは、参照とのサブタイピング（共変的な方法で）という意味でのみサブタイピングを持ちます。

例：

```move
let x: &u64 = &0;
let y: &mut u64 = &mut 1;

// (&u64, &mut u64) is a subtype of (&u64, &u64)
// since &mut u64 is a subtype of &u64
let (a, b): (&u64, &u64) = (x, y);

// (&mut u64, &mut u64) is a subtype of (&u64, &u64)
// since &mut u64 is a subtype of &u64
let (c, d): (&u64, &u64) = (y, y);

// highlight-error-start
// ERROR! (&u64, &mut u64) is NOT a subtype of (&mut u64, &mut u64)
// since &u64 is NOT a subtype of &mut u64
let (e, f): (&mut u64, &mut u64) = (x, y);
// highlight-error-end
```

## 所有権

上記で述べたように、タプル値は実行時には実際には存在しません。そして現在、このためローカル変数に格納することはできません（ただし、この機能は将来的に追加される可能性があります）。そのため、現在タプルは移動のみ可能です。なぜなら、コピーするにはまずローカル変数に格納する必要があるからです。
