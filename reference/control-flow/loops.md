---
title: 'ループ | リファレンス'
description: ''
---

# Moveのループ構文

多くのプログラムは値の反復を必要とし、Moveはこれらの状況でコードを書くことを可能にする`while`と`loop`形式を提供します。さらに、`break`（ループを終了する）と`continue`（この反復の残りをスキップして制御フロー構造のトップに戻る）を使用して、実行中にこれらのループの制御フローを変更することもできます。

## `while`ループ

`while`構文は、条件（`bool`型の式）が`false`と評価されるまで、本体（`unit`型の式）を繰り返します。

以下は、`1`から`n`までの数値の合計を計算する簡単な`while`ループの例です：

```move
fun sum(n: u64): u64 {
    let mut sum = 0;
    let mut i = 1;
    while (i <= n) {
        sum = sum + i;
        i = i + 1
    };

    sum
}
```

無限`while`ループも許可されています：

```move
fun foo() {
    while (true) { }
}
```

> より簡潔で読みやすい目的を達成するために、ループの代わりにマクロを使用する方が良い方法です。
> この記事では、上記の関数`sum`を例として、マクロ関数の魅力を体験します：
```move
fun sum(n: u64): u64 {
    vector::tabulate!(n, |i| i + 1).fold!(0, |sum, num| sum + num)
}
```

### `while`ループ内での`break`の使用

Moveでは、`while`ループは`break`を使用して早期に終了できます。例えば、ベクター内の値の位置を探していて、見つかったら`break`したいとします：

```move
fun find_position(values: &vector<u64>, target_value: u64): Option<u64> {
    let size = values.length();
    let mut i = 0;
    let mut found = false;

    while (i < size) {
        if (values[i] == target_value) {
            found = true;
            break
        };
        i = i + 1
    };

    if (found) {
        option::some(i)
    } else {
        option::none<u64>()
    }
}
```

ここで、借用されたベクター値がターゲット値と等しい場合、`found`フラグを`true`に設定し、その後`break`を呼び出します。これにより、プログラムはループを終了します。

最後に、`while`ループの`break`は値を取ることができないことに注意してください：`while`ループは常にユニット型`()`を返すため、`break`も同様です。

### `while`ループ内での`continue`の使用

`break`と同様に、Moveの`while`ループは`continue`を呼び出してループ本体の一部をスキップできます。これにより、条件が満たされない場合に計算の一部をスキップできます。以下の例のように：

```move
fun sum_even(values: &vector<u64>): u64 {
    let size = values.length();
    let mut i = 0;
    let mut even_sum = 0;

    while (i < size) {
        let number = values[i];
        i = i + 1;
        if (number % 2 == 1) continue;
        even_sum = even_sum + number;
    };
    even_sum
}
```

このコードは、提供されたベクターを反復します。各エントリについて、そのエントリが偶数の場合、`even_sum`に追加します。そうでない場合は、`continue`を呼び出し、合計操作をスキップして`while`ループの条件チェックに戻ります。

## `loop`式

`loop`式は、`break`に達するまでループ本体（`()`型の式）を繰り返します：

```move
fun sum(n: u64): u64 {
    let mut sum = 0;
    let mut i = 1;

    loop {
       i = i + 1;
       if (i >= n) break;
       sum = sum + i;
    };

    sum
}
```

`break`がない場合、ループは永遠に続きます。以下の例では、`loop`に`break`がないため、プログラムは永遠に実行されます：

```move
fun foo() {
    let mut i = 0;
    loop { i = i + 1 }
}
```

### `loop`での値を使った`break`の使用

常に`()`を返す`while`ループとは異なり、`loop`は`break`を使用して値を返すことができます。これにより、`loop`式全体がその型の値に評価されます。例えば、上記の`find_position`を`loop`と`break`を使用して書き直し、見つかったらすぐにインデックスを返すことができます：

```move
fun find_position(values: &vector<u64>, target_value: u64): Option<u64> {
    let size = values.length();
    let mut i = 0;

    loop {
        if (values[i] == target_value) {
            break option::some(i)
        } else if (i >= size) {
            break option::none()
        };
        i = i + 1;
    }
}
```

このループはオプション結果でブレークし、関数本体の最後の式として、その値を最終的な関数結果として生成します。

### `loop`式内での`continue`の使用

予想されるように、`continue`は`loop`内でも使用できます。以下は、`while`の代わりに`break`と`continue`を使用した`loop`で書き直された前の`sum_even`関数です。

```move
fun sum_even(values: &vector<u64>): u64 {
    let size = values.length();
    let mut i = 0;
    let mut even_sum = 0;

    loop {
        if (i >= size) break;
        let number = values[i];
        i = i + 1;
        if (number % 2 == 1) continue;
        even_sum = even_sum + number;
    };
    even_sum
}
```

## `while`と`loop`の型

Moveでは、ループは型付き式です。`while`式は常に型`()`を持ちます。

```move
let () = while (i < 10) { i = i + 1 };
```

`loop`に`break`が含まれている場合、式はブレークの型を持ちます。値のないブレークはユニット型`()`を持ちます。

```move
(loop { if (i < 10) i = i + 1 else break }: ());
let () = loop { if (i < 10) i = i + 1 else break };

let x: u64 = loop { if (i < 10) i = i + 1 else break 5 };
let x: u64 = loop { if (i < 10) { i = i + 1; continue} else break 5 };
```

さらに、ループに複数のブレークが含まれている場合、それらはすべて同じ型を返す必要があります：

```move
// invalid -- first break returns (), second returns 5
let x: u64 = loop { if (i < 10) break else break 5 };
```

`loop`に`break`がない場合、`loop`は`return`、`abort`、`break`、`continue`と同様に任意の型を持つことができます。

```move
(loop (): u64);
(loop (): address);
(loop (): &vector<vector<u8>>);
```

ネストしたループから抜けるなど、より正確な制御フローが必要な場合は、次の章でMoveでのラベル付き制御フローの使用について説明します。
