---
title: 'ラベル付き制御フロー | リファレンス'
description: ''
---

# ラベル付き制御フロー

Moveは、ループとコードブロックの両方を書く際にラベル付き制御フローをサポートし、ループの`break`と`continue`、ブロックからの`return`を可能にします（これはマクロの存在下で特に有用です）。

## ループ

ループを使用すると、関数内の特定のラベルを定義し、制御を転送できます。例えば、2つのループをネストし、それらのラベルで`break`と`continue`を使用して、制御フローを正確に指定できます。任意の`loop`または`while`形式の前に`'label:`形式を付けることで、そこから直接ブレークまたはコンティニューできます。

この動作を実演するために、ネストした数値ベクター（つまり`vector<vector<u64>>`）を受け取り、閾値に対して合計する関数を考えてみましょう。この関数は以下のように動作します：

- すべての数値の合計が閾値を下回る場合、その合計を返します。
- 現在の合計に数値を追加すると閾値を超える場合、現在の合計を返します。

これを、ベクターのベクターをネストしたループとして反復し、外側のループにラベルを付けることで書くことができます。内側のループで任意の加算が閾値を超える場合、外側のラベルで`break`を使用して両方のループを一度に抜けることができます：

```move
fun sum_until_threshold(input: &vector<vector<u64>>, threshold: u64): u64 {
    let mut sum = 0;
    let mut i = 0;
    let input_size = input.length();

    'outer: loop {
        // breaks to outer since it is the closest enclosing loop
        if (i >= input_size) break sum;

        let vec = &input[i];
        let size = vec.length();
        let mut j = 0;

        while (j < size) {
            let v_entry = vec[j];
            if (sum + v_entry < threshold) {
                sum = sum + v_entry;
            } else {
                // the next element we saw would break the threshold,
                // so we return the current sum
                break 'outer sum
            };
            j = j + 1;
        };
        i = i + 1;
    }
}
```

これらの種類のラベルは、ネストしたループ形式でも使用でき、より大きなコード本体で正確な制御を提供します。例えば、各エントリが反復を必要とする大きなテーブルを処理していて、内側または外側のループを継続する可能性がある場合、ラベルを使用してそのコードを表現できます：

```move
let x = 'outer: loop {
    ...
    'inner: while (cond) {
        ...
        if (cond0) { break 'outer value };
        ...
        if (cond1) { continue 'inner }
        else if (cond2) { continue 'outer }
        ...
    }
        ...
};
```

> ループの代わりにマクロを使用する方が良い方法です。同様に、`return`を使用してフローを制御します。
> 上記の関数`sum_until_threshold`のように、`macro`を使用して書き直すことができます：
```move
fun sum_until_threshold(input: &vector<vector<u64>>, threshold: u64): u64 {
    'outer: {
        (*input).fold!(0, |sum, inner_vec| {
            inner_vec.fold!(sum, |sum, num| if (sum + num < threshold) sum + num else return 'outer sum)
        })
    }
}
```

## ラベル付きブロック

ラベル付きブロックを使用すると、マクロラムダ内や値の返却を含む、関数内の非ローカル制御フローを含むMoveプログラムを書くことができます：

```move
fun named_return(n: u64): vector<u8> {
    let x = 'a: {
        if (n % 2 == 0) {
            return 'a b"even"
        };
        b"odd"
    };
    x
}
```

この簡単な例では、プログラムは入力`n`が偶数かどうかをチェックします。偶数の場合、プログラムは`'a:`というラベルが付いたブロックを値`b"even"`で抜けます。そうでない場合、コードは続行し、`'a:`というラベルが付いたブロックを値`b"odd"`で終了します。最後に、`x`をその値に設定してから返します。

この制御フロー機能は、マクロ本体全体でも機能します。例えば、ベクター内の最初の偶数を見つける関数を書きたいとし、ベクター要素をループで反復する`for_ref`というマクロがあるとします：

```move
macro fun for_ref<$T>($vs: &vector<$T>, $f: |&$T|) {
    let vs = $vs;
    let mut i = 0;
    let end = vs.length();
    while (i < end) {
        $f(vs.borrow(i));
        i = i + 1;
    }
}
```

`for_ref`とラベルを使用して、ループを抜けて最初に見つけた偶数を返す`for_ref`に渡すラムダ式を書くことができます：

```move
fun find_first_even(vs: vector<u64>): Option<u64> {
    'result: {
        for_ref!(&vs, |n| if (*n % 2 == 0) { return 'result option::some(*n)});
        option::none()
    }
}
```

この関数は、偶数を見つけるまで`vs`を反復し、それ（または偶数が存在しない場合は`option::none()`）を返します。これにより、名前付きラベルは`for!`などの制御フローマクロと相互作用する強力なツールとなり、それらのコンテキストで反復動作をカスタマイズできます。

## 制限

プログラムの動作を明確にするため、`break`と`continue`はループラベルでのみ使用でき、`return`はブロックラベルでのみ機能します。この目的のために、以下のプログラムはエラーを生成します：

```move
fun bad_loop() {
    'name: loop {
        return 'name 5
            // ^^^^^ Invalid usage of 'return' with a loop block label
    }
}

fun bad_block() {
    'name: {
        continue 'name;
              // ^^^^^ Invalid usage of 'break' with a loop block label
        break 'name;
           // ^^^^^ Invalid usage of 'break' with a loop block label
    }
}
```
