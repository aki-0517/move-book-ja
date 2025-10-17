---
title: 'ローカル変数とスコープ | リファレンス'
description: ''
---

# ローカル変数とスコープ

Moveのローカル変数は字句的（静的）にスコープされます。新しい変数は`let`キーワードで導入され、同じ名前の以前のローカル変数をシャドウします。`mut`とマークされたローカル変数は可変であり、直接的に、または可変参照を通じて更新できます。

## ローカル変数の宣言

### `let`バインディング

Moveプログラムは`let`を使用して変数名を値にバインドします：

```move
let x = 1;
let y = x + x;
```

`let`は値をローカル変数にバインドせずに使用することもできます。

```move
let x;
```

ローカル変数は後で値を代入することができます。

```move
let x;
if (cond) {
  x = 1
} else {
  x = 0
}
```

これは、デフォルト値を提供できない場合にループから値を抽出しようとする際に非常に便利です。

```move
let x;
let mut i = 0;
loop {
    let (res, cond) = foo(i);
    if (!cond) {
        x = res;
        break
    };
    i = i + 1;
}
```

ローカル変数を代入_後_に変更したり、可変参照（`&mut`）として借用したりするには、`mut`として宣言する必要があります。

```move
let mut x = 0;
if (cond) x = x + 1;
foo(&mut x);
```

詳細については、以下の[代入](#assignments)セクションを参照してください。

### 変数は使用前に代入する必要があります

Moveの型システムは、ローカル変数が代入される前に使用されることを防ぎます。

```move
let x;
// highlight-error
x + x // ERROR! xが代入される前に使用されています
```

```move
let x;
if (cond) x = 0;
// highlight-error
x + x // ERROR! xはすべてのケースで値を持たない
```

```move
let x;
while (cond) x = 0;
// highlight-error
x + x // ERROR! xはすべてのケースで値を持たない
```

### 有効な変数名

変数名にはアンダースコア`_`、`a`から`z`の文字、`A`から`Z`の文字、および`0`から`9`の数字を含めることができます。変数名はアンダースコア`_`または`a`から`z`の文字で始まる必要があります。大文字で始めることは_できません_。

```move
// すべて有効
let x = e;
let _x = e;
let _A = e;
let x0 = e;
let xA = e;
let foobar_123 = e;

// すべて無効
// highlight-error-start
let X = e; // ERROR!
let Foo = e; // ERROR!
// highlight-error-end
```

### 型注釈

ローカル変数の型は、Moveの型システムによってほぼ常に推論できます。ただし、Moveでは可読性、明確性、またはデバッグのために有用な明示的な型注釈を許可しています。型注釈を追加する構文は以下の通りです：

```move
let x: T = e; // "型Tの変数xは式eで初期化される"
```

明示的な型注釈の例：

```move
module 0::example;

public struct S { f: u64, g: u64 }

fun annotated() {
    let u: u8 = 0;
    let b: vector<u8> = b"hello";
    let a: address = @0x0;
    let (x, y): (&u64, &mut u64) = (&0, &mut 1);
    let S { f, g: f2 }: S = S { f: 0, g: 1 };
}
```

型注釈は常にパターンの右側に配置する必要があることに注意してください：

```move
// highlight-error-start
// ERROR! let (x, y): (&u64, &mut u64) = ... である必要があります
let (x: &u64, y: &mut u64) = (&0, &mut 1);
// highlight-error-end
```

### 注釈が必要な場合

型システムが型を推論できない場合、ローカル型注釈が必要になることがあります。これは、ジェネリック型の型引数を推論できない場合によく発生します。例えば：

```move
// highlight-error-start
let _v1 = vector[]; // ERROR!
//        ^^^^^^^^ この型を推論できませんでした。注釈を追加してください
// highlight-error-end
let v2: vector<u64> = vector[]; // エラーなし
```

より稀なケースでは、型システムが発散コード（その後のすべてのコードが到達不可能なコード）の型を推論できない場合があります。[`return`](./functions#return-expression)と[`abort`](./abort-and-assert)はどちらも式であり、任意の型を持つことができます。[`loop`](./control-flow/loops)は`break`がある場合は型`()`を持ちます（または`break e`がある場合、`e: T`なら型`T`を持ちます）が、`loop`からのbreakがない場合は任意の型を持つことができます。これらの型が推論できない場合、型注釈が必要です。例えば、このコード：

```move
let a: u8 = return ();
let b: bool = abort 0;
let c: signer = loop ();

// highlight-error-start
let x = return (); // ERROR!
//  ^ この型を推論できませんでした。注釈を追加してください
let y = abort 0; // ERROR!
//  ^ この型を推論できませんでした。注釈を追加してください
let z = loop (); // ERROR!
//  ^ この型を推論できませんでした。注釈を追加してください
// highlight-error-end
```

このコードに型注釈を追加すると、デッドコードや未使用のローカル変数に関する他のエラーが露呈されますが、この例はこの問題を理解するのに依然として役立ちます。

### タプルを使用した複数宣言

`let`はタプルを使用して一度に複数のローカル変数を導入できます。括弧内で宣言されたローカル変数は、タプルからの対応する値で初期化されます。

```move
let () = ();
let (x0, x1) = (0, 1);
let (y0, y1, y2) = (0, 1, 2);
let (z0, z1, z2, z3) = (0, 1, 2, 3);
```

式の型はタプルパターンのアリティと正確に一致する必要があります。

```move
// highlight-error
let (x, y) = (0, 1, 2); // ERROR!
// highlight-error
let (x, y, z, q) = (0, 1, 2); // ERROR!
```

単一の`let`で同じ名前の複数のローカル変数を宣言することはできません。

```move
// highlight-error
let (x, x) = 0; // ERROR!
```

宣言されるローカル変数の可変性は混在させることができます。

```move
let (mut x, y) = (0, 1);
x = 1;
```

### 構造体を使用した複数宣言

`let`は構造体を分解（またはマッチング）する際に、一度に複数のローカル変数を導入することもできます。この形式では、`let`は構造体のフィールドの値で初期化されるローカル変数のセットを作成します。構文は以下のようになります：

```move
public struct T { f1: u64, f2: u64 }
```

```move
let T { f1: local1, f2: local2 } = T { f1: 1, f2: 2 };
// local1: u64
// local2: u64
```

位置構造体でも同様です

```move
public struct P(u64, u64)
```

そして

```move
let P (local1, local2) = P ( 1, 2 );
// local1: u64
// local2: u64
```

より複雑な例を以下に示します：

```move
module 0::example;

public struct X(u64)
public struct Y { x1: X, x2: X }

fun new_x(): X {
    X(1)
}

fun example() {
    let Y { x1: X(f), x2 } = Y { x1: new_x(), x2: new_x() };
    assert!(f + x2.0 == 2, 42);

    let Y { x1: X(f1), x2: X(f2) } = Y { x1: new_x(), x2: new_x() };
    assert!(f1 + f2 == 2, 42);

    // `struct X`は`drop`アビリティを持たず、手動で破棄する必要があります
    let X(_) = x2;
}
```

構造体のフィールドは二重の役割を果たし、バインドするフィールドを識別し_かつ_変数の名前を識別します。これは時々パニングと呼ばれます。

```move
let Y { x1, x2 } = e;
```

これは以下と同等です：

```move
let Y { x1: x1, x2: x2 } = e;
```

タプルで示されたように、単一の`let`で同じ名前の複数のローカル変数を宣言することはできません。

```move
// highlight-error
let Y { x1: x, x2: x } = e; // ERROR!
```

そしてタプルと同様に、宣言されるローカル変数の可変性は混在させることができます。

```move
let Y { x1: mut x1, x2 } = e;
```

さらに、可変性の注釈はパニングされたフィールドに適用することができます。同等の例を示すと

```move
let Y { mut x1, x2 } = e;
```

### 参照に対する分解

上記の構造体の例では、letでバインドされた値はムーブされ、構造体の値を破棄してそのフィールドをバインドします。

```move
public struct T { f1: u64, f2: u64 }
```

```move
let T { f1: local1, f2: local2 } = T { f1: 1, f2: 2 };
// local1: u64
// local2: u64
```

このシナリオでは、構造体の値`T { f1: 1, f2: 2 }`は`let`の後にはもう存在しません。

代わりに構造体の値をムーブして破棄したくない場合は、その各フィールドを借用することができます。例えば：

```move
let t = T { f1: 1, f2: 2 };
let T { f1: local1, f2: local2 } = &t;
// local1: &u64
// local2: &u64
```

同様に可変参照でも：

```move
let mut t = T { f1: 1, f2: 2 };
let T { f1: local1, f2: local2 } = &mut t;
// local1: &mut u64
// local2: &mut u64
```

This behavior can also work with nested structs.

```move
module 0::example;

public struct X(u64)
public struct Y { x1: X, x2: X }

fun new_x(): X {
    X(1)
}

fun example() {
    let mut y = Y { x1: new_x(), x2: new_x() };

    let Y { x1: X(f), x2 } = &y;
    assert!(*f + x2.0 == 2, 42);

    let Y { x1: X(f1), x2: X(f2) } = &mut y;
    *f1 = *f1 + 1;
    *f2 = *f2 + 1;
    assert!(*f1 + *f2 == 4, 42);

    // `struct X and struct Y` without `drop` ability and needs to be destroyed manually
    let Y { x1: X(_), x2: X(_) } = y;
}
```

### 値の無視

`let`バインディングでは、一部の値を無視することがしばしば役立ちます。`_`で始まるローカル変数は無視され、新しい変数を導入しません。

```move
fun three(): (u64, u64, u64) {
    (0, 1, 2)
}
```

```move
let (x1, _, z1) = three();
let (x2, _y, z2) = three();
assert!(x1 + z1 == x2 + z2, 42);
```

コンパイラは未使用のローカル変数に対して警告を発するため、時々これが必要になることがあります。

```move
let (x1, y, z1) = three(); // WARNING!
//       ^ 未使用のローカル 'y'
```

### 一般的な`let`文法

`let`のすべての異なる構造を組み合わせることができます！これにより、`let`文の一般的な文法に到達します：

> _let-binding_ → **let** _pattern-or-list_ _type-annotation_<sub>_opt_</sub> >
> _initializer_<sub>_opt_</sub> > _pattern-or-list_ → _pattern_ | **(** _pattern-list_ **)** >
> _pattern-list_ → _pattern_ **,**<sub>_opt_</sub> | _pattern_ **,** _pattern-list_ >
> _type-annotation_ → **:** _type_ _initializer_ → **=** _expression_

バインディングを導入する項目の一般的な用語は_パターン_です。パターンはデータを分解し（可能であれば再帰的に）、バインディングを導入する役割を果たします。パターンの文法は以下の通りです：

> _pattern_ -> _local-variable_ | _struct-type_ **\{** _field-binding-list_ **\}** >
> _field-binding-list_ → _field-binding_ **,**<sub>_opt_</sub> | _field-binding_ **,** >
> _field-binding-list_ > _field-binding_ → _field_ | _field_ **:** _pattern_

A few concrete examples with this grammar applied:

```move
    let (x, y): (u64, u64) = (0, 1);
//       ^                           local-variable
//       ^                           pattern
//          ^                        local-variable
//          ^                        pattern
//          ^                        pattern-list
//       ^^^^                        pattern-list
//      ^^^^^^                       pattern-or-list
//            ^^^^^^^^^^^^           type-annotation
//                         ^^^^^^^^  initializer
//  ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ let-binding

    let Foo { f, g: x } = Foo { f: 0, g: 1 };
//      ^^^                                    struct-type
//            ^                                field
//            ^                                field-binding
//               ^                             field
//                  ^                          local-variable
//                  ^                          pattern
//               ^^^^                          field-binding
//            ^^^^^^^                          field-binding-list
//      ^^^^^^^^^^^^^^^                        pattern
//      ^^^^^^^^^^^^^^^                        pattern-or-list
//                      ^^^^^^^^^^^^^^^^^^^^   initializer
//  ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ let-binding
```

## 変更

### 代入

ローカル変数が導入された後（`let`または関数パラメータとして）、`mut`ローカル変数は代入を通じて変更することができます：

```move
x = e
```

`let`バインディングとは異なり、代入は式です。一部の言語では代入は代入された値を返しますが、Moveでは任意の代入の型は常に`()`です。

```move
(x = e: ())
```

実用的には、代入が式であることは、ブレース（`{`...`}`）で新しい式ブロックを追加せずに使用できることを意味します。

```move
let x;
if (cond) x = 1 else x = 2;
```

The assignment uses the similar pattern syntax scheme as `let` bindings, but with absence of `mut`:

```move
module 0::example;

public struct X { f: u64 }

fun new_x(): X {
    X { f: 1 }
}

// Note: this example will complain about unused variables and assignments.
fun example() {
    let (mut x, mut y, mut f, mut g) = (0, 0, 0, 0);

    (X { f }, X { f: x }) = (new_x(), new_x());
    assert!(f + x == 2, 42);

    (x, y, f, _, g) = (0, 0, 0, 0, 0);
}
```

Note that a local variable can only have one type, so the type of the local cannot change between
assignments.

```move
let mut x;
x = 0;
// highlight-error
x = false; // ERROR!
```

### Mutating through a reference

In addition to directly modifying a local with assignment, a `mut` local can be modified via a
mutable reference `&mut`.

```move
let mut x = 0;
let r = &mut x;
*r = 1;
assert!(x == 1, 42);
```

This is particularly useful if either:

(1) You want to modify different variables depending on some condition.

```move
let mut x = 0;
let mut y = 1;
let r = if (cond) &mut x else &mut y;
*r = *r + 1;
```

(2) You want another function to modify your local value.

```move
let mut x = 0;
modify_ref(&mut x);
```

This sort of modification is how you modify structs and vectors!

```move
let mut v = vector[];
vector::push_back(&mut v, 100);
assert!(*vector::borrow(&v, 0) == 100, 42);
```

For more details, see [Move references](./primitive-types/references).

## スコープ

`let`で宣言されたローカル変数は、_そのスコープ内_の任意の後続式で使用できます。スコープは式ブロック`{`...`}`で宣言されます。

ローカル変数は宣言されたスコープの外では使用できません。

```move
let x = 0;
{
    let y = 1;
};
// highlight-error-start
x + y // ERROR!
//  ^ アンバインドローカル 'y'
// highlight-error-end
```

しかし、外側スコープのローカル変数はネストしたスコープで使用_できます_。

```move
{
    let x = 0;
    {
        let y = x + 1; // 有効
    }
}
```

Locals can be mutated in any scope where they are accessible. That mutation survives with the local,
regardless of the scope that performed the mutation.

```move
let mut x = 0;
x = x + 1;
assert!(x == 1, 42);
{
    x = x + 1;
    assert!(x == 2, 42);
};
assert!(x == 2, 42);
```

### Expression Blocks

An expression block is a series of statements separated by semicolons (`;`). The resulting value of
an expression block is the value of the last expression in the block.

```move
{ let x = 1; let y = 1; x + y }
```

In this example, the result of the block is `x + y`.

A statement can be either a `let` declaration or an expression. Remember that assignments (`x = e`)
are expressions of type `()`.

```move
{ let x; let y = 1; x = 1; x + y }
```

Function calls are another common expression of type `()`. Function calls that modify data are
commonly used as statements.

```move
{ let v = vector[]; vector::push_back(&mut v, 1); v }
```

This is not just limited to `()` types---any expression can be used as a statement in a sequence!

```move
{
    let x = 0;
    x + 1; // value is discarded
    x + 2; // value is discarded
    b"hello"; // value is discarded
}
```

But! If the expression contains a resource (a value without the `drop` [ability](./abilities)), you
will get an error. This is because Move's type system guarantees that any value that is dropped has
the `drop` [ability](./abilities). (Ownership must be transferred or the value must be explicitly
destroyed within its declaring module.)

```move
{
    let x = 0;
// highlight-error-start
    Coin { value: x }; // ERROR!
//  ^^^^^^^^^^^^^^^^^ unused value without the `drop` ability
// highlight-error-end
    x
}
```

If a final expression is not present in a block---that is, if there is a trailing semicolon `;`,
there is an implicit [unit `()` value](https://en.wikipedia.org/wiki/Unit_type). Similarly, if the
expression block is empty, there is an implicit unit `()` value.

Both are equivalent

```move
{ x = x + 1; 1 / x; }
```

```move
{ x = x + 1; 1 / x; () }
```

Similarly both are equivalent

```move
{ }
```

```move
{ () }
```

An expression block is itself an expression and can be used anyplace an expression is used. (Note:
The body of a function is also an expression block, but the function body cannot be replaced by
another expression.)

```move
let my_vector: vector<vector<u8>> = {
    let mut v = vector[];
    vector::push_back(&mut v, b"hello");
    vector::push_back(&mut v, b"goodbye");
    v
};
```

(The type annotation is not needed in this example and only added for clarity.)

### シャドウイング

`let`が既にスコープ内にある名前のローカル変数を導入する場合、以前の変数はこのスコープの残りの部分でアクセスできなくなります。これを_シャドウイング_と呼びます。

```move
let x = 0;
assert!(x == 0, 42);

let x = 1; // xはシャドウされる
assert!(x == 1, 42);
```

ローカル変数がシャドウされた場合、以前と同じ型を保持する必要はありません。

```move
let x = 0;
assert!(x == 0, 42);

let x = b"hello"; // xはシャドウされる
assert!(x == b"hello", 42);
```

After a local is shadowed, the value stored in the local still exists, but will no longer be
accessible. This is important to keep in mind with values of types without the
[`drop` ability](./abilities), as ownership of the value must be transferred by the end of the
function.

```move
module 0::example;

public struct Coin has store { value: u64 }

fun unused_coin(): Coin {
// highlight-error-start
    let x = Coin { value: 0 }; // ERROR!
//      ^ This local still contains a value without the `drop` ability
    x.value = 1;
    let x = Coin { value: 10 };
    x
//  ^ Invalid return
// highlight-error-end
}
```

When a local is shadowed inside a scope, the shadowing only remains for that scope. The shadowing is
gone once that scope ends.

```move
let x = 0;
{
    let x = 1;
    assert!(x == 1, 42);
};
assert!(x == 0, 42);
```

Remember, locals can change type when they are shadowed.

```move
let x = 0;
{
    let x = b"hello";
    assert!(x == b"hello", 42);
};
assert!(x == 0, 42);
```

## ムーブとコピー

Moveのすべてのローカル変数は、`move`または`copy`のどちらかの方法で使用できます。どちらかが指定されていない場合、Moveコンパイラは`copy`または`move`のどちらを使用するかを推論できます。これは、上記のすべての例で、コンパイラによって`move`または`copy`が挿入されることを意味します。ローカル変数は`move`または`copy`を使用せずに使用することはできません。

`copy`は他のプログラミング言語から来た人にとって最も馴染み深いと感じられるでしょう。これは、その式で使用するために変数内の値の新しいコピーを作成します。`copy`では、ローカル変数を複数回使用できます。

```move
let x = 0;
let y = copy x + 1;
let z = copy x + 2;
```

`copy`[アビリティ](./abilities)を持つ任意の値はこの方法でコピーでき、`move`が指定されない限り暗黙的にコピーされます。

`move`はデータをコピー_せずに_ローカル変数から値を取り出します。`move`が発生した後、たとえ値の型が`copy`[アビリティ](./abilities)を持っていても、ローカル変数は使用不可能になります。

```move
let x = 1;
// highlight-error-start
let y = move x + 1;
//      ------ ローカルはここでムーブされました
let z = move x + 2; // Error!
//      ^^^^^^ ローカル 'x' の無効な使用
// highlight-error-end
y + z
```

### Safety

Move's type system will prevent a value from being used after it is moved. This is the same safety
check described in [`let` declaration](#let-bindings) that prevents local variables from being used
before it is assigned a value.

<!-- For more information, see TODO future section on ownership and move semantics. -->

### Inference

As mentioned above, the Move compiler will infer a `copy` or `move` if one is not indicated. The
algorithm for doing so is quite simple:

- Any value with the `copy` [ability](./abilities) is given a `copy`.
- Any reference (both mutable `&mut` and immutable `&`) is given a `copy`.
  - Except under special circumstances where it is made a `move` for predictable borrow checker
    errors. This will happen once the reference is no longer used.
- Any other value is given a `move`.

Given the structs

```move
public struct Foo has copy, drop, store { f: u64 }
public struct Coin has store { value: u64 }
```

we have the following example

```move
let s = b"hello";
let foo = Foo { f: 0 };
let coin = Coin { value: 0 };
let coins = vector[Coin { value: 0 }, Coin { value: 0 }];

let s2 = s; // copy
let foo2 = foo; // copy
let coin2 = coin; // move
let coins2 = coins; // move

let x = 0;
let b = false;
let addr = @0x42;
let x_ref = &x;
let coin_ref = &mut coin2;

let x2 = x; // copy
let b2 = b; // copy
let addr2 = @0x42; // copy
let x_ref2 = x_ref; // copy
let coin_ref2 = coin_ref; // copy
```
