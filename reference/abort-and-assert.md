---
title: 'Abort と Assert | リファレンス'
description: ''
---

# Abort と Assert

[`return`](./functions)と`abort`は実行を終了する2つの制御フロー構造で、1つは現在の関数用、もう1つはトランザクション全体用です。

[`return`についての詳細情報はリンクされたセクションで確認できます](./functions#return-expression)

## `abort`

`abort`は引数を取らないか、1つだけ引数を取る式です。その引数は`u64`型の**abortコード**です。例えば：

```move
abort
abort 42
```

`abort`式は現在の関数の実行を停止し、現在のトランザクションによって状態に加えられたすべての変更を元に戻します（ただし、この保証はMoveの特定のデプロイメントのアダプターによって維持される必要があります）。`abort`を「キャッチ」したり、その他の方法で処理するメカニズムはありません。

幸い、Moveではトランザクションはオール・オア・ナッシングです。つまり、ストレージへの変更はトランザクションが成功した場合にのみ一度にすべて行われます。Suiでは、これはオブジェクトが変更されないことを意味します。

この変更のトランザクショナルなコミットのため、abortの後で変更をロールバックすることを心配する必要はありません。このアプローチは柔軟性に欠けますが、非常にシンプルで予測可能です。

[`return`](./functions)と同様に、`abort`は何らかの条件が満たされない場合に制御フローを終了するのに便利です。

この例では、関数はベクターから2つのアイテムをポップしますが、ベクターに2つのアイテムがない場合は早期にabortします

<!-- {{#include ../../packages/reference/sources/abort-and-assert.move}} -->

```move
fun pop_twice<T>(v: &mut vector<T>): (T, T) {
    if (v.length() < 2) abort 42;
    (v.pop_back(), v.pop_back())
}
```

これは制御フロー構造の深い部分でさらに便利です。例えば、この関数はベクター内のすべての数値が指定された`bound`より小さいことをチェックし、そうでなければabortします

```move
fun check_vec(v: &vector<u64>, bound: u64) {
    let mut i = 0;
    let n = v.length();
    while (i < n) {
        let cur = v[i];
        if (cur > bound) abort 42;
        i = i + 1;
    }
}
```

> `macro`と`abort`の組み合わせ：

```move
fun check_vec(v: &vector<u64>, bound: u64) {
    v.do_ref!(|num| if (*num > bound) abort 42);
}
```

### `assert`

`assert`はMoveコンパイラによって提供される組み込みのマクロ操作です。2つの引数を取ります：`bool`型の条件と`u64`型のコードです

```move
assert!(condition: bool, code: u64)
```

この操作はマクロなので、`!`で呼び出す必要があります。これは`assert`への引数が式で呼び出されることを示しています。つまり、`assert`は通常の関数ではなく、バイトコードレベルでは存在しません。コンパイラ内で以下のように置き換えられます

```move
if (condition) () else abort code
```

`assert`は単独の`abort`よりも一般的に使用されます。上記の`abort`の例は`assert`を使って書き直すことができます

```move
fun pop_twice<T>(v: &mut vector<T>): (T, T) {
    assert!(v.length() >= 2, 42); // Now uses 'assert'
    (v.pop_back(), v.pop_back())
}
```

そして

```move
fun check_vec(v: &vector<u64>, bound: u64) {
    let mut i = 0;
    let n = v.length();
    while (i < n) {
        let cur = v[i];
        assert!(cur <= bound, 42); // Now uses 'assert'
        i = i + 1;
    }
}
```

> `macro`と`assert`の組み合わせ：

```move
fun check_vec(v: &vector<u64>, bound: u64) {
    v.do_ref!(|num| assert!(*num <= bound, 42));
}
```

操作がこの`if-else`で置き換えられるため、`code`の引数が常に評価されるわけではないことに注意してください。例えば：

```move
assert!(true, 1 / 0)
```

これは算術エラーを引き起こしません。これは以下と同等です

```move
if (true) () else abort (1 / 0)
```

つまり、算術式は決して評価されません！

### Move VMのAbortコード

`abort`を使用する際、VMが`u64`コードをどのように使用するかを理解することが重要です。

通常、成功した実行後、Move VMと特定のデプロイメントのアダプターがストレージに加えられた変更を決定します。

`abort`に達した場合、VMは代わりにエラーを示します。そのエラーには2つの情報が含まれます：

- abortを生成したモジュール（パッケージ/アドレス値とモジュール名）
- abortコード

例えば

```move
module 0x2::example {
    public fun aborts() {
        abort 42
    }
}

module 0x3::invoker {
    public fun always_aborts() {
        0x2::example::aborts()
    }
}
```

上記の`always_aborts`関数のようなトランザクションが`0x2::example::aborts`を呼び出した場合、VMはモジュール`0x2::example`とコード`42`を示すエラーを生成します。

これは、モジュール内で複数のabortをグループ化するのに便利です。

この例では、モジュールは複数の関数で使用される2つの別々のエラーコードを持っています

```move
module 0::example;

use std::vector;

const EEmptyVector: u64 = 0;
const EIndexOutOfBounds: u64 = 1;

// iをjに、jをkに、kをiに移動
public fun rotate_three<T>(v: &mut vector<T>, i: u64, j: u64, k: u64) {
    let n = v.length();
    assert!(n > 0, EEmptyVector);
    assert!(i < n, EIndexOutOfBounds);
    assert!(j < n, EIndexOutOfBounds);
    assert!(k < n, EIndexOutOfBounds);

    v.swap(i, k);
    v.swap(j, k);
}

public fun remove_twice<T>(v: &mut vector<T>, i: u64, j: u64): (T, T) {
    let n = v.length();
    assert!(n > 0, EEmptyVector);
    assert!(i < n, EIndexOutOfBounds);
    assert!(j < n, EIndexOutOfBounds);
    assert!(i > j, EIndexOutOfBounds);

    (v.remove(i), v.remove(j))
}
```

## `abort`の型

`abort i`式は任意の型を持つことができます！これは、両方の構造が通常の制御フローから抜け出すため、その型の値に評価される必要がないからです。

以下は有用ではありませんが、型チェックは通ります

```move
let y: address = abort 0;
```

この動作は、一部のブランチでは値を生成するが、すべてではない分岐命令がある状況で役立ちます。例えば：

```move
let b =
    if (x == 0) false
    else if (x == 1) true
    else abort 42;
//       ^^^^^^^^ `abort 42`は`bool`型を持ちます
```
