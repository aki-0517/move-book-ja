---
title: '構造体 | リファレンス'
description: ''
---

# 構造体とリソース

_構造体_は型付きフィールドを含むユーザー定義のデータ構造です。構造体は、他の構造体を含む、参照でない、タプルでない任意の型を格納できます。

構造体は、すべての「アセット」値または制限のない値を定義するために使用でき、それらの値に対して実行される操作は構造体の[アビリティ](./abilities)によって制御できます。デフォルトでは、構造体は線形であり一時的です。これは、コピーできない、ドロップできない、ストレージに保存できないことを意味します。これは、すべての値が所有権を転送され（線形）、プログラムの実行終了までに値を処理する必要がある（一時的）ことを意味します。構造体に[アビリティ](./abilities)を付与することで、値をコピーまたはドロップしたり、ストレージに保存したり、ストレージスキーマを定義したりできるように、この動作を緩和できます。

## 構造体の定義

構造体はモジュール内で定義する必要があり、構造体のフィールドは名前付きまたは位置指定のどちらかであることができます：

```move
module a::m;

public struct Foo { x: u64, y: bool }
public struct Bar {}
public struct Baz { foo: Foo, }
//                          ^ 注意: 末尾のカンマがあっても問題ありません

public struct PosFoo(u64, bool)
public struct PosBar()
public struct PosBaz(Foo)
```

構造体は再帰的にできないため、以下の定義は無効です：

```move
public struct Foo { x: Foo }
//                     ^ ERROR! 再帰定義

public struct A { b: B }
public struct B { a: A }
//                   ^ ERROR! 再帰定義

public struct D(D)
//              ^ ERROR! 再帰定義
```

### 可視性

お気づきかもしれませんが、すべての構造体は`public`として宣言されています。これは、構造体の型を他のモジュールから参照できることを意味します。ただし、構造体のフィールドと、構造体を作成または破棄する能力は、構造体を定義するモジュールの内部にとどまります。

将来的には、[関数](./functions#visibility)と同様に、構造体を`public(package)`または内部として宣言する機能を追加する予定です。

### アビリティ

上記で述べたように：デフォルトでは、構造体宣言は線形で一時的です。したがって、値をこれらの方法で使用できるようにする（例：コピー、ドロップ、[オブジェクト](./abilities/object)に保存、または保存可能な[オブジェクト](./abilities/object)を定義するために使用）ために、構造体に`has <ability>`で注釈を付けることで[アビリティ](./abilities)を付与できます：

```move
module a::m {
    public struct Foo has copy, drop { x: u64, y: bool }
}
```

アビリティ宣言は構造体のフィールドの前または後に配置できます。しかし、どちらか一方のみを使用でき、両方は使用できません。構造体のフィールドの後に宣言する場合、アビリティ宣言はセミコロンで終了する必要があります：

```move
module a::m;

public struct PreNamedAbilities has copy, drop { x: u64, y: bool }
public struct PostNamedAbilities { x: u64, y: bool } has copy, drop;
public struct PostNamedAbilitiesInvalid { x: u64, y: bool } has copy, drop
//                                                                        ^ エラー！セミコロンが不足

public struct NamedInvalidAbilities has copy { x: u64, y: bool } has drop;
//                                                               ^ エラー！重複するアビリティ宣言

public struct PrePositionalAbilities has copy, drop (u64, bool)
public struct PostPositionalAbilities (u64, bool) has copy, drop;
public struct PostPositionalAbilitiesInvalid (u64, bool) has copy, drop
//                                                                     ^ エラー！セミコロンが不足
public struct InvalidAbilities has copy (u64, bool) has drop;
//                                                  ^ エラー！重複するアビリティ宣言
```

詳細については、[構造体のアビリティの注釈](./abilities#annotating-structs-and-enums)のセクションを参照してください。

### 命名

構造体は大文字`A`から`Z`で始まる必要があります。最初の文字の後、構造体名には
アンダースコア`_`、文字`a`から`z`、文字`A`から`Z`、または数字`0`から`9`を含めることができます。

```move
public struct Foo {}
public struct BAR {}
public struct B_a_z_4_2 {}
public struct P_o_s_Foo()
```

`A`から`Z`で始まるこの命名制限は、将来の言語機能の余地を与えるために設けられています。
後で削除される可能性があります。

## 構造体の使用

### 構造体の作成

構造体型の値は、構造体名を示し、その後に各フィールドの値を指定することで作成（または「パック」）できます。

名前付きフィールドを持つ構造体の場合、フィールドの順序は関係ありませんが、フィールド名を提供する必要があります。位置指定フィールドを持つ構造体の場合、フィールドの順序は構造体定義内のフィールドの順序と一致する必要があり、パラメータを囲むために`{}`の代わりに`()`を使用して作成する必要があります。

```move
module a::m;

public struct Foo has drop { x: u64, y: bool }
public struct Baz has drop { foo: Foo }
public struct Positional(u64, bool) has drop;

fun example() {
    let foo = Foo { x: 0, y: false };
    let baz = Baz { foo: foo };
    // Note: positional struct values are created using parentheses and
    // based on position instead of name.
    let pos = Positional(0, false);
    let pos_invalid = Positional(false, 0);
    //                           ^ ERROR! Fields are out of order and the types don't match.
}
```

For structs with named fields, you can use the following shorthand if you have a local variable with
the same name as the field:

```move
let baz = Baz { foo: foo };
// is equivalent to
let baz = Baz { foo };
```

これは時々「フィールド名パニング」と呼ばれます。

### パターンマッチングによる構造体の破棄

構造体の値は、構築するのと同様の構文を使用してパターンでバインドまたは代入することで破棄できます。

```move
module a::m;

public struct Foo { x: u64, y: bool }
public struct Bar(Foo)
public struct Baz {}
public struct Qux()

fun example_destroy_foo() {
    let foo = Foo { x: 3, y: false };
    let Foo { x, y: foo_y } = foo;
    //        ^ shorthand for `x: x`

    // two new bindings
    //   x: u64 = 3
    //   foo_y: bool = false
}

fun example_destroy_foo_wildcard() {
    let foo = Foo { x: 3, y: false };
    let Foo { x, y: _ } = foo;

    // only one new binding since y was bound to a wildcard
    //   x: u64 = 3
}

fun example_destroy_foo_assignment() {
    let x: u64;
    let y: bool;
    Foo { x, y } = Foo { x: 3, y: false };

    // mutating existing variables x and y
    //   x = 3, y = false
}

fun example_foo_ref() {
    let foo = Foo { x: 3, y: false };
    let Foo { x, y } = &foo;

    // two new bindings
    //   x: &u64
    //   y: &bool
}

fun example_foo_ref_mut() {
    let foo = Foo { x: 3, y: false };
    let Foo { x, y } = &mut foo;

    // two new bindings
    //   x: &mut u64
    //   y: &mut bool
}

fun example_destroy_bar() {
    let bar = Bar(Foo { x: 3, y: false });
    let Bar(Foo { x, y }) = bar;
    //            ^ nested pattern

    // two new bindings
    //   x: u64 = 3
    //   y: bool = false
}

fun example_destroy_baz() {
    let baz = Baz {};
    let Baz {} = baz;
}

fun example_destroy_qux() {
    let qux = Qux();
    let Qux() = qux;
}
```

### 構造体フィールドへのアクセス

構造体のフィールドはドット演算子`.`を使用してアクセスできます。

名前付きフィールドを持つ構造体の場合、フィールドは名前でアクセスできます：

```move
public struct Foo { x: u64, y: bool }
let foo = Foo { x: 3, y: true };
let x = foo.x;  // x == 3
let y = foo.y;  // y == true
```

位置指定構造体の場合、フィールドは構造体定義内の位置でアクセスできます：

```move
public struct PosFoo(u64, bool)
let pos_foo = PosFoo(3, true);
let x = pos_foo.0;  // x == 3
let y = pos_foo.1;  // y == true
```

構造体フィールドを借用またはコピーせずにアクセスすることは、フィールドのアビリティ制約の対象となります。
詳細については、[構造体とフィールドの借用](#borrowing-structs-and-fields)と
[フィールドの読み書き](#reading-and-writing-fields)のセクションを参照してください。

### 構造体とフィールドの借用

`&`と`&mut`演算子を使用して、構造体またはフィールドへの参照を作成できます。これらの例には、
操作の型を示すためのいくつかのオプションの型注釈（例：`: &Foo`）が含まれています。

```move
let foo = Foo { x: 3, y: true };
let foo_ref: &Foo = &foo;
let y: bool = foo_ref.y;         // reading a field via a reference to the struct
let x_ref: &u64 = &foo.x;        // borrowing a field by extending a reference to the struct

let x_ref_mut: &mut u64 = &mut foo.x;
*x_ref_mut = 42;            // modifying a field via a mutable reference
```

It is possible to borrow inner fields of nested structs:

```move
let foo = Foo { x: 3, y: true };
let bar = Bar(foo);

let x_ref = &bar.0.x;
```

You can also borrow a field via a reference to a struct:

```move
let foo = Foo { x: 3, y: true };
let foo_ref = &foo;
let x_ref = &foo_ref.x;
// this has the same effect as let x_ref = &foo.x
```

### フィールドの読み書き

フィールドの値を読み取り、コピーする必要がある場合は、借用されたフィールドを逆参照できます：

```move
let foo = Foo { x: 3, y: true };
let bar = Bar(copy foo);
let x: u64 = *&foo.x;
let y: bool = *&foo.y;
let foo2: Foo = *&bar.0;
```

より標準的には、ドット演算子を使用して借用なしで構造体のフィールドを読み取ることができます。
[逆参照](./primitive-types/references#reading-and-writing-through-references)と同様に、
フィールド型は`copy`[アビリティ](./abilities)を持つ必要があります。

```move
let foo = Foo { x: 3, y: true };
let x = foo.x;  // x == 3
let y = foo.y;  // y == true
```

ドット演算子をチェーンしてネストしたフィールドにアクセスできます：

```move
let bar = Bar(Foo { x: 3, y: true });
let x = baz.0.x; // x = 3;
```

しかし、これはベクターや他の構造体などの非プリミティブ型を含むフィールドでは許可されません：

```move
let foo = Foo { x: 3, y: true };
let bar = Bar(foo);
let foo2: Foo = *&bar.0;
let foo3: Foo = bar.0; // エラー！*&で明示的なコピーを追加する必要があります
```

構造体のフィールドを可変借用して新しい値を代入できます：

```move
let mut foo = Foo { x: 3, y: true };
*&mut foo.x = 42;     // foo = Foo { x: 42, y: true }
*&mut foo.y = !foo.y; // foo = Foo { x: 42, y: false }
let mut bar = Bar(foo);               // bar = Bar(Foo { x: 42, y: false })
*&mut bar.0.x = 52;                   // bar = Bar(Foo { x: 52, y: false })
*&mut bar.0 = Foo { x: 62, y: true }; // bar = Bar(Foo { x: 62, y: true })
```

逆参照と同様に、代わりにドット演算子を直接使用してフィールドを変更できます。どちらの場合も、
フィールド型は`drop`[アビリティ](./abilities)を持つ必要があります。

```move
let mut foo = Foo { x: 3, y: true };
foo.x = 42;     // foo = Foo { x: 42, y: true }
foo.y = !foo.y; // foo = Foo { x: 42, y: false }
let mut bar = Bar(foo);         // bar = Bar(Foo { x: 42, y: false })
bar.0.x = 52;                   // bar = Bar(Foo { x: 52, y: false })
bar.0 = Foo { x: 62, y: true }; // bar = Bar(Foo { x: 62, y: true })
```

代入のドット構文は、構造体への参照を通しても機能します：

```move
let mut foo = Foo { x: 3, y: true };
let foo_ref = &mut foo;
foo_ref.x = foo_ref.x + 1;
```

## 特権構造体操作

構造体型`T`に対するほとんどの構造体操作は、`T`を宣言するモジュール内でのみ実行できます：

- 構造体型は、構造体を定義するモジュール内でのみ作成（「パック」）または破棄（「アンパック」）できます。
- 構造体のフィールドは、構造体を定義するモジュール内でのみアクセス可能です。

これらのルールに従って、モジュール外で構造体を変更したい場合は、それらのためのパブリックAPIを提供する必要があります。章の最後に、これの例がいくつか含まれています。

ただし、[上記の可視性セクション](#visibility)で述べたように、構造体_型_は常に他のモジュールから見えます

```move
module a::m {
    public struct Foo has drop { x: u64 }

    public fun new_foo(): Foo {
        Foo { x: 42 }
    }
}

module a::n {
    use a::m::Foo;

    public struct Wrapper has drop {
        foo: Foo
        //   ^ valid the type is public

    }

    fun f1(foo: Foo) {
        let x = foo.x;
        //      ^ ERROR! cannot access fields of `Foo` outside of `a::m`
    }

    fun f2() {
        let foo_wrapper = Wrapper { foo: a::m::new_foo() };
        //                               ^ valid the function is public
    }
}

```

## 所有権

[構造体の定義](#defining-structs)で上記で述べたように、構造体はデフォルトで線形で一時的です。
これは、コピーまたはドロップできないことを意味します。この特性は、お金のような現実世界の資産を
モデル化する際に非常に有用です。お金が複製されたり、流通で失われたりすることを望まないからです。

```move
module a::m;

public struct Foo { x: u64 }

public fun copying() {
    let foo = Foo { x: 100 };
    let foo_copy = copy foo; // ERROR! 'copy'-ing requires the 'copy' ability
    let foo_ref = &foo;
    let another_copy = *foo_ref // ERROR! dereference requires the 'copy' ability
}

public fun destroying_1() {
    let foo = Foo { x: 100 };

    // error! when the function returns, foo still contains a value.
    // This destruction requires the 'drop' ability
}

public fun destroying_2(f: &mut Foo) {
    *f = Foo { x: 100 } // error!
                        // destroying the old value via a write requires the 'drop' ability
}
```

例`fun destroying_1`を修正するには、値を手動で「アンパック」する必要があります：

```move
module a::m;

public struct Foo { x: u64 }

public fun destroying_1_fixed() {
    let foo = Foo { x: 100 };
    let Foo { x: _ } = foo;
}
```

構造体を分解できるのは、それが定義されているモジュール内のみであることを思い出してください。これは、
システム内の特定の不変条件（例：お金の保存）を強制するために活用できます。

一方で、構造体が価値のあるものを表していない場合は、`copy`と`drop`のアビリティを追加して、
他のプログラミング言語からより馴染みのある構造体値を得ることができます：

```move
module a::m;

public struct Foo has copy, drop { x: u64 }

public fun run() {
    let foo = Foo { x: 100 };
    let foo_copy = foo;
    //             ^ this code copies foo,
    //             whereas `let x = move foo` would move foo

    let x = foo.x;            // x = 100
    let x_copy = foo_copy.x;  // x = 100

    // both foo and foo_copy are implicitly discarded when the function returns
}
```

## Storage

Structs can be used to define storage schemas, but the details are different per deployment of Move.
See the documentation for the [`key` ability](./abilities#key) and
[Sui objects](./abilities/object) for more details.
