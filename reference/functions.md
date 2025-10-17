---
title: '関数 | リファレンス'
description: ''
---

# 関数

関数はモジュール内で宣言され、モジュールのロジックと動作を定義します。関数は、他の関数から呼び出されるか、実行のエントリポイントとして再利用できます。

## 宣言

関数は`fun`キーワードに続いて、関数名、型パラメータ、パラメータ、戻り値の型、そして最後に関数本体で宣言されます。

```text
<visibility>? <entry>? <macro>? fun <identifier><[type_parameters: constraint],*>([identifier: type],*): <return_type> <function_body>
```

例えば

```move
fun foo<T1, T2>(x: u64, y: T1, z: T2): (T2, T1, u64) { (z, y, x) }
```

### 可視性

モジュール関数は、デフォルトでは同じモジュール内でのみ呼び出すことができます。これらの内部（時々プライベートと呼ばれる）関数は、他のモジュールからやエントリポイントとして呼び出すことはできません。

```move
module a::m {
    fun foo(): u64 { 0 }
    fun calls_foo(): u64 { foo() } // 有効
}

module b::other {
    fun calls_m_foo(): u64 {
        a::m::foo() // ERROR!
//      ^^^^^^^^^^^ 'foo'は'a::m'の内部関数です
    }
}
```

他のモジュールからのアクセスを許可するには、関数を`public`または`public(package)`として宣言する必要があります。可視性に関連して、[`entry`](#entry-modifier)関数は実行のエントリポイントとして呼び出すことができます。

#### `public`可視性

`public`関数は_任意_のモジュールに定義された_任意_の関数から呼び出すことができます。以下の例で示されるように、`public`関数は以下から呼び出すことができます：

- 同じモジュールで定義された他の関数
- 別のモジュールで定義された関数
- 実行のエントリポイントとして

```move
module a::m {
    public fun foo(): u64 { 0 }
    fun calls_foo(): u64 { foo() } // 有効
}

module b::other {
    fun calls_m_foo(): u64 {
        a::m::foo() // 有効
    }
}
```

実行エントリポイントの詳細については[以下のセクション](#entry-modifier)を参照してください。

#### `public(package)`可視性

`public(package)`可視性修飾子は、関数が使用できる場所についてより細かい制御を提供するための、`public`修飾子のより制限的な形式です。`public(package)`関数は以下から呼び出すことができます：

- 同じモジュールで定義された他の関数
- 同じパッケージ（同じアドレス）で定義された他の関数

```move
module a::m {
    public(package) fun foo(): u64 { 0 }
    fun calls_foo(): u64 { foo() } // 有効
}

module a::n {
    fun calls_m_foo(): u64 {
        a::m::foo() // 有効、同じく`a`内です
    }
}

module b::other {
    fun calls_m_foo(): u64 {
        a::m::foo() // ERROR!
//      ^^^^^^^^^^^ 'foo'は`a`のモジュールからのみ呼び出すことができます
    }
}
```

#### 非推奨 `public(friend)`可視性

`public(package)`の追加前は、`public(friend)`が同じパッケージ内の関数への限定的なパブリックアクセスを許可するために使用されていましたが、許可されたモジュールのリストを呼び出されるモジュールが明示的に列挙する必要がありました。詳細は[Friends](./friends)を参照してください。

### `entry`修飾子

`public`関数に加えて、モジュール内に実行のエントリポイントとして使用したい関数がある場合があります。`entry`修飾子は、他のモジュールに機能を公開することなく、モジュール関数が実行を開始できるように設計されています。

本質的に、`public`と`entry`関数の組み合わせはモジュールの「main」関数を定義し、Moveプログラムが実行を開始できる場所を指定します。

ただし、`entry`関数は_依然として_他のMove関数から呼び出すことができることに注意してください。つまり、Moveプログラムの開始点として機能_できます_が、そのケースに制限されるわけではありません。

例えば：

```move
module a::m {
    entry fun foo(): u64 { 0 }
    fun calls_foo(): u64 { foo() } // 有効！
}

module a::n {
    fun calls_m_foo(): u64 {
        a::m::foo() // ERROR!
//      ^^^^^^^^^^^ 'foo'は'a::m'の内部関数です
    }
}
```

`entry`関数はパラメータや戻り値の型に制限がある場合があります。ただし、これらの制限はMoveの個々のデプロイに固有です。

[Suiでの`entry`関数のドキュメントはこちらでご確認いただけます。](https://docs.sui.io/concepts/sui-move-concepts/entry-functions)

テストを簡単にするために、`entry`関数は[`#[test]`と`#[test_only]`](./unit-testing)コンテキストから呼び出すことができます。

```move
module a::m {
    entry fun foo(): u64 { 0 }
}
module a::m_test {
    #[test]
    fun my_test(): u64 { a::m::foo() } // 有効！
    #[test_only]
    fun my_test_helper(): u64 { a::m::foo() } // 有効！
}
```

### `macro`修飾子

通常の関数とは異なり、`macro`関数は実行時に存在しません。代わりに、これらの関数はコンパイル時に各呼び出し箇所でインライン置換されます。これらの`macro`関数は、このコンパイルプロセスを活用して、高階の_ラムダ_スタイルの関数を引数として受け取るなど、標準関数を超えた機能を提供します。コンパイル時に展開されるこれらのラムダ引数により、関数本体の一部を引数としてマクロに渡すことができます。例えば、以下の単純なループマクロを考えてみてください。ここではループ本体がラムダとして提供されます：

```move
macro fun n_times($n: u64, $body: |u64| -> ()) {
    let n = $n;
    let mut i = 0;
    while (i < n) {
        $body(i);
        i = i + 1;
    }
}

fun example() {
    let mut sum = 0;
    n_times!(10, |x| sum = sum + x );
}
```

詳細については、[マクロ](./functions/macros)の章を参照してください。

### 名前

関数名は`a`から`z`の文字で始まることができます。最初の文字の後、関数名にはアンダースコア`_`、`a`から`z`の文字、`A`から`Z`の文字、または`0`から`9`の数字を含めることができます。

```move
fun fOO() {}
fun bar_42() {}
fun bAZ_19() {}
```

### 型パラメータ

名前の後、関数は型パラメータを持つことができます。

```move
fun id<T>(x: T): T { x }
fun example<T1: copy, T2>(x: T1, y: T2): (T1, T1, T2) { (copy x, x, y) }
```

詳細については、[Moveジェネリックス](./generics)を参照してください。

### パラメータ

関数パラメータは、ローカル変数名の後に型注釈を続けて宣言します。

```move
fun add(x: u64, y: u64): u64 { x + y }
```

これは`x`が型`u64`を持つと読みます。

関数はパラメータを全く持たなくても構いません。

```move
fun useless() { }
```

これは新しいまたは空のデータ構造を作成する関数で非常に一般的です。

```move
module a::example;

public struct Counter { count: u64 }

fun new_counter(): Counter {
    Counter { count: 0 }
}
```

### 戻り値の型

パラメータの後、関数はその戻り値の型を指定します。

```move
fun zero(): u64 { 0 }
```

ここで`: u64`は関数の戻り値の型が`u64`であることを示しています。

[タプル](./primitive-types/tuples)を使用して、関数は複数の値を返すことができます：

```move
fun one_two_three(): (u64, u64, u64) { (0, 1, 2) }
```

戻り値の型が指定されていない場合、関数は暗黙的に単位型`()`の戻り値の型を持ちます。これらの関数は同等です：

```move
fun just_unit(): () { () }
fun just_unit() { () }
fun just_unit() { }
```

[タプルセクション](./primitive-types/tuples)で言及されたように、これらのタプル「値」はランタイム値としては存在しません。これは、単位型`()`を返す関数は実行中に任意の値を返さないことを意味します。

### 関数本体

関数の本体は式ブロックです。関数の戻り値はシーケンスの最後の値です。

```move
fun example(): u64 {
    let mut x = 0;
    x = x + 1;
    x // 'x'を返す
}
```

戻り値の詳細については[以下のセクション](#returning-values)を参照してください。

式ブロックの詳細については、[Move変数](./variables)を参照してください。

### ネイティブ関数

一部の関数は本体が指定されておらず、代わりにVMによって本体が提供されます。これらの関数は`native`とマークされています。

VMのソースコードを変更せずに、プログラマーは新しいネイティブ関数を追加することはできません。さらに、`native`関数は標準ライブラリコードまたは特定のMove環境に必要な機能のために使用されることが意図されています。

おそらく目にすることになるほとんどの`native`関数は、`vector`などの標準ライブラリコードにあります。

```move
module std::vector {
    native public fun length<Element>(v: &vector<Element>): u64;
    ...
}
```

## 呼び出し

関数を呼び出すとき、名前はエイリアスまたは完全修飾した名前で指定できます。

```move
module a::example {
    public fun zero(): u64 { 0 }
}

module b::other {
    use a::example::{Self, zero};
    fun call_zero() {
        // 上記の`use`で、これらの呼び出しはすべて同等です
        a::example::zero();
        example::zero();
        zero();
    }
}
```

関数を呼び出すときは、すべてのパラメータに引数を渡す必要があります。

```move
module a::example {
    public fun takes_none(): u64 { 0 }
    public fun takes_one(x: u64): u64 { x }
    public fun takes_two(x: u64, y: u64): u64 { x + y }
    public fun takes_three(x: u64, y: u64, z: u64): u64 { x + y + z }
}

module b::other {
    fun call_all() {
        a::example::takes_none();
        a::example::takes_one(0);
        a::example::takes_two(0, 1);
        a::example::takes_three(0, 1, 2);
    }
}
```

型引数は指定するか推論させるかのどちらかです。両方の呼び出しは同等です。

```move
module a::example {
    public fun id<T>(x: T): T { x }
}

module b::other {
    fun call_all() {
        a::example::id(0);
        a::example::id<u64>(0);
    }
}
```

詳細については、[Moveジェネリックス](./generics)を参照してください。

## 値の返し方

関数の結果、つまり「戻り値」は、関数本体の最終値です。例えば

```move
fun add(x: u64, y: u64): u64 {
    x + y
}
```

ここでの戻り値は`x + y`の結果です。

[上記で言及したように](#function-body)、関数の本体は[式ブロック](./variables)です。式ブロックはさまざまな文をシーケンスでき、ブロック内の最後の式がそのブロックの値になります。

```move
fun double_and_add(x: u64, y: u64): u64 {
    let double_x = x * 2;
    let double_y = y * 2;
    double_x + double_y
}
```

ここでの戻り値は`double_x + double_y`の結果です。

### `return`式

関数はその本体が評価される値を暗黙的に返します。ただし、関数は明示的な`return`式を使用することもできます：

```move
fun f1(): u64 { return 0 }
fun f2(): u64 { 0 }
```

これら二つの関数は同等です。もう少し複雑な例では、関数は二つの`u64`値を減算しますが、二番目の値が大きすぎる場合は`0`で早期リターンします：

```move
fun safe_sub(x: u64, y: u64): u64 {
    if (y > x) return 0;
    x - y
}
```

この関数の本体は`if (y > x) 0 else x - y`として書くこともできたことに注意してください。

しかし、`return`が真に輝くのは、他の制御フロー構造の深い部分からの終了です。この例では、関数はベクターを反復して、指定された値のインデックスを見つけます：

```move
fun index_of<T>(v: &vector<T>, target: &T): Option<u64> {
    let mut i = 0;
    let n = v.length();
    while (i < n) {
        if (&v[i] == target) return option::some(i);
        i = i + 1
    };

    option::none()
}
```

引数なしで`return`を使用するのは`return ()`のショートハンドです。つまり、以下の二つの関数は同等です：

```move
fun foo() { return }
fun foo() { return () }
```
