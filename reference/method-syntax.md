---
title: 'メソッド構文 | リファレンス'
description: ''
---

# メソッド

構文的な利便性として、Moveの一部の関数は値の「メソッド」として呼び出すことができます。これは、関数を呼び出すために`.`演算子を使用することで行われ、`.`の左側の値が関数の最初の引数（時々レシーバーと呼ばれる）です。その値の型が静的にどの関数が呼び出されるかを決定します。これは、この構文が動的呼び出しを示す可能性がある他の言語との重要な違いです。動的呼び出しでは、呼び出される関数が実行時に決定されます。Moveでは、すべての関数呼び出しが静的に決定されます。

要するに、この構文は`use`でエイリアスを作成する必要がなく、関数の最初の引数を明示的に借用する必要がなく、関数を呼び出すことを容易にするために存在します。さらに、これは関数呼び出しに必要なボイラープレートの量を減らし、関数呼び出しをチェーンすることを容易にするため、コードをより読みやすくすることができます。

## 構文

メソッドを呼び出す構文は以下の通りです：

```text
<expression> . <identifier> <[type_arguments],*> ( <arguments> )
```

例

```move
coin.value();
*nums.borrow_mut(i) = 5;
```

## メソッド解決

メソッドが呼び出されると、コンパイラはレシーバー（`.`の左側の引数）の型に基づいて、どの関数が呼び出されるかを静的に決定します。コンパイラは、型とメソッド名から呼び出されるべきモジュールと関数名へのマッピングを維持します。このマッピングは、現在スコープ内の`use fun`エイリアスと、レシーバー型の定義モジュールの適切な関数から作成されます。すべての場合において、レシーバー型は関数の最初の引数であり、値渡しまたは参照渡しのいずれかです。

このセクションでは、メソッドが関数に「解決」されると言うとき、コンパイラがメソッドを通常の[関数](./functions)呼び出しで静的に置き換えることを意味します。例えば、`x.foo(e)`で`foo`が`a::m::foo`に解決される場合、コンパイラは`x.foo(e)`を`a::m::foo(x, e)`に置き換え、必要に応じて`x`を[自動借用](#automatic-borrowing)します。

### Functions in the Defining Module

In a type’s defining module, the compiler will automatically create a method alias for any function
型が関数の最初の引数である場合の型の宣言。例えば、

```move
module a::m;

public struct X() has copy, drop, store;
public fun foo(x: &X) { ... }
public fun bar(flag: bool, x: &X) { ... }
```

関数`foo`は、型`X`の値に対してメソッドとして呼び出すことができます。しかし、最初の引数ではない（`bool`はそのモジュールで定義されていないため、`bool`用のものは作成されません）。例えば、

```move
fun example(x: a::m::X) {
    x.foo(); // 有効
    // x.bar(true); エラー！
}
```

### `use fun`エイリアス

従来の[`use`](uses)と同様に、`use fun`ステートメントは現在のスコープにローカルなエイリアスを作成します。これは現在のモジュールまたは現在の式ブロック用である可能性があります。しかし、エイリアスは型に関連付けられます。

`use fun`ステートメントの構文は以下の通りです：

```move
use fun <function> as <type>.<method alias>;
```

これは`<function>`のエイリアスを作成し、`<type>`は`<method alias>`として受け取ることができます。

例

```move
module a::cup;

public struct Cup<T>(T) has copy, drop, store;

public fun cup_borrow<T>(c: &Cup<T>): &T {
    &c.0
}

public fun cup_value<T>(c: Cup<T>): T {
    let Cup(t) = c;
    t
}

public fun cup_swap<T: drop>(c: &mut Cup<T>, t: T) {
    c.0 = t;
}
```

これらの関数に対して`use fun`エイリアスを作成できます

```move
module b::example;

use fun a::cup::cup_borrow as Cup.borrow;
use fun a::cup::cup_value as Cup.value;
use fun a::cup::cup_swap as Cup.set;

fun example(c: &mut Cup<u64>) {
    let _ = c.borrow(); // a::cup::cup_borrowに解決
    let v = c.value(); // a::cup::cup_valueに解決
    c.set(v * 2); // a::cup::cup_swapに解決
}
```

`use fun`の`<function>`は完全に解決されたパスである必要はなく、代わりにエイリアスを使用できることに注意してください。したがって、上記の例の宣言は以下のように同等に書くことができます

```move
use a::cup::{Self, cup_swap};

use fun cup::cup_borrow as Cup.borrow;
use fun cup::cup_value as Cup.value;
use fun cup_swap as Cup.set;
```

これらの例は現在のモジュールの関数をリネームするのに便利ですが、この機能は他のモジュールの型にメソッドを宣言するのにより有用かもしれません。例えば、`Cup`に新しいユーティリティを追加したい場合、`use fun`エイリアスを使用してメソッド構文を引き続き使用できます

```move
module b::example;

fun double(c: &Cup<u64>): Cup<u64> {
    let v = c.value();
    Cup::new(v * 2)
}
```

通常、`b::example`が`Cup`を定義していないため、`double(&c)`として呼び出す必要がありますが、代わりに`use fun`エイリアスを使用できます

```move
fun double_double(c: Cup<u64>): (Cup<u64>, Cup<u64>) {
    use fun b::example::double as Cup.dub;
    (c.dub(), c.dub()) // 両方の呼び出しでb::example::doubleに解決
}
```

`use fun`は任意のスコープで作成できますが、`use fun`の対象`<function>`は`<type>`と同じ最初の引数を持たなければなりません。

```move
public struct X() has copy, drop, store;

fun new(): X { X() }
fun flag(flag: bool): u8 { if (flag) 1 else 0 }

use fun new as X.new; // エラー！
use fun flag as X.flag; // エラー！
// `new`も`flag`も型`X`の最初の引数を持たない
```

しかし、`<type>`の任意の最初の引数を使用でき、参照と可変参照も含まれます

```move
public struct X() has copy, drop, store;

public fun by_val(_: X) {}
public fun by_ref(_: &X) {}
public fun by_mut(_: &mut X) {}

// すべて有効、任意のスコープで
use fun by_val as X.v;
use fun by_ref as X.r;
use fun by_mut as X.m;
```

ジェネリクスについては、メソッドはジェネリック型の_すべての_インスタンスに関連付けられることに注意してください。インスタンス化に応じて異なる関数に解決するようにメソッドをオーバーロードすることはできません。

```move
public struct Cup<T>(T) has copy, drop, store;

public fun value<T: copy>(c: &Cup<T>): T {
    c.0
}

use fun value as Cup<bool>.flag; // エラー！
use fun value as Cup<u64>.num; // エラー！
// どちらの場合も、`use fun`エイリアスはジェネリックにできず、型のすべてのインスタンスで動作する必要があります
```

### `public use fun` Aliases

Unlike a traditional [`use`](uses), the `use fun` statement can be made `public`, which allows it
to be used outside of its declared scope. A `use fun` can be made `public` if it is declared in the
module that defines the receivers type, much like the method aliases that are
[automatically created](#functions-in-the-defining-module) for functions in the defining module. Or
conversely, one can think that an implicit `public use fun` is created automatically for every
function in the defining module that has a first argument of the receiver type (if it is defined in
that module). Both of these views are equivalent.

```move
module a::cup;

public struct Cup<T>(T) has copy, drop, store;

public use fun cup_borrow as Cup.borrow;
public fun cup_borrow<T>(c: &Cup<T>): &T {
    &c.0
}
```

In this example, a public method alias is created for `a::cup::Cup.borrow` and
`a::cup::Cup.cup_borrow`. Both resolve to `a::cup::cup_borrow`. And both are "public" in the sense
that they can be used outside of `a::cup`, without an additional `use` or `use fun`.

```move
module b::example;

fun example<T: drop>(c: a::cup::Cup<u64>) {
    c.borrow(); // resolves to a::cup::cup_borrow
    c.cup_borrow(); // resolves to a::cup::cup_borrow
}
```

The `public use fun` declarations thus serve as a way of renaming a function if you want to give it
a cleaner name for use with method syntax. This is especially helpful if you have a module with
multiple types, and similarly named functions for each type.

```move
module a::shapes;

public struct Rectangle { base: u64, height: u64 }
public struct Box { base: u64, height: u64, depth: u64 }

// Rectangle and Box can have methods with the same name

public use fun rectangle_base as Rectangle.base;
public fun rectangle_base(rectangle: &Rectangle): u64 {
    rectangle.base
}

public use fun box_base as Box.base;
public fun box_base(box: &Box): u64 {
    box.base
}
```

Another use for `public use fun` is adding methods to types from other modules. This can be helpful
in conjunction with functions spread out across a single package.

```move
module a::cup {
    public struct Cup<T>(T) has copy, drop, store;

    public fun new<T>(t: T): Cup<T> { Cup(t) }
    public fun borrow<T>(c: &Cup<T>): &T {
        &c.0
    }
    // `public use fun` to a function defined in another module
    public use fun a::utils::split as Cup.split;
}

module a::utils {
    use a::m::{Self, Cup};

    public fun split<u64>(c: Cup<u64>): (Cup<u64>, Cup<u64>) {
        let Cup(t) = c;
        let half = t / 2;
        let rem = if (t > 0) t - half else 0;
        (cup::new(half), cup::new(rem))
    }

}
```

And note that this `public use fun` does not create a circular dependency, as the `use fun` is not
present after the module is compiled--all methods are resolved statically.

### Interactions with `use` Aliases

A small detail to note is that method aliases respect normal `use` aliases.

```move
module a::cup {
    public struct Cup<T>(T) has copy, drop, store;

    public fun cup_borrow<T>(c: &Cup<T>): &T {
        &c.0
    }
}

module b::other {
    use a::cup::{Cup, cup_borrow as borrow};

    fun example(c: &Cup<u64>) {
        c.borrow(); // resolves to a::cup::cup_borrow
    }
}
```

A helpful way to think about this is that `use` creates an implicit `use fun` alias for the function
whenever it can. In this case the `use a::cup::cup_borrow as borrow` creates an implicit
`use fun a::cup::cup_borrow as Cup.borrow` because it would be a valid `use fun` alias. Both views
are equivalent. This line of reasoning can inform how specific methods will resolve with shadowing.
See the cases in [Scoping](#scoping) for more details.

### Scoping

If not `public`, a `use fun` alias is local to its scope, much like a normal [`use`](uses). For
example

```move
module a::m {
    public struct X() has copy, drop, store;
    public fun foo(_: &X) {}
    public fun bar(_: &X) {}
}

module b::other {
    use a::m::X;

    use fun a::m::foo as X.f;

    fun example(x: &X) {
        x.f(); // resolves to a::m::foo
        {
            use a::m::bar as f;
            x.f(); // resolves to a::m::bar
        };
        x.f(); // still resolves to a::m::foo
        {
            use fun a::m::bar as X.f;
            x.f(); // resolves to a::m::bar
        }
    }
```

## Automatic Borrowing

When resolving a method, the compiler will automatically borrow the receiver if the function expects
a reference. For example

```move
module a::m;

public struct X() has copy, drop;
public fun by_val(_: X) {}
public fun by_ref(_: &X) {}
public fun by_mut(_: &mut X) {}

fun example(mut x: X) {
    x.by_ref(); // resolves to a::m::by_ref(&x)
    x.by_mut(); // resolves to a::m::by_mut(&mut x)
}
```

In these examples, `x` was automatically borrowed to `&x` and `&mut x` respectively. This will also
work through field access

```move
module a::m;

public struct X() has copy, drop;
public fun by_val(_: X) {}
public fun by_ref(_: &X) {}
public fun by_mut(_: &mut X) {}

public struct Y has drop { x: X }

fun example(mut y: Y) {
    y.x.by_ref(); // resolves to a::m::by_ref(&y.x)
    y.x.by_mut(); // resolves to a::m::by_mut(&mut y.x)
}
```

Note that in both examples, the local variable had to be labeled as [`mut`](./variables) to allow
for the `&mut` borrow. Without this, there would be an error saying that `x` (or `y` in the second
example) is not mutable.

Keep in mind that without a reference, normal rules for variable and field access come into play.
Meaning a value might be moved or copied if it is not borrowed.

```move
module a::m;

public struct X() has copy, drop;
public fun by_val(_: X) {}
public fun by_ref(_: &X) {}
public fun by_mut(_: &mut X) {}

public struct Y has drop { x: X }
public fun drop_y(y: Y) { y }

fun example(y: Y) {
    y.x.by_val(); // copies `y.x` since `by_val` is by-value and `X` has `copy`
    y.drop_y(); // moves `y` since `drop_y` is by-value and `Y` does _not_ have `copy`
}
```

## Chaining

Method calls can be chained, because any expression can be the receiver of the method.

```move
module a::shapes {
    public struct Point has copy, drop, store { x: u64, y: u64 }
    public struct Line has copy, drop, store { start: Point, end: Point }

    public fun x(p: &Point): u64 { p.x }
    public fun y(p: &Point): u64 { p.y }

    public fun start(l: &Line): &Point { &l.start }
    public fun end(l: &Line): &Point { &l.end }

}

module b::example {
    use a::shapes::Line;

    public fun x_values(l: Line): (u64, u64) {
        (l.start().x(), l.end().x())
    }

}
```

In this example for `l.start().x()`, the compiler first resolves `l.start()` to
`a::shapes::start(&l)`. Then `.x()` is resolved to `a::shapes::x(a::shapes::start(&l))`. Similarly
for `l.end().x()`. Keep in mind, this feature is not "special"--the left-hand side of the `.` can be
any expression, and the compiler will resolve the method call as normal. We simply draw attention to
this sort of "chaining" because it is a common practice to increase readability.
