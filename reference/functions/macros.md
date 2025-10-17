---
title: 'マクロ関数 | リファレンス'
description: ''
---

# マクロ関数

マクロ関数は、各呼び出しサイトでコンパイル時に展開される関数を定義する方法です。マクロの引数は通常の関数のように熱心に評価されるのではなく、式によって置換されます。さらに、呼び出し元は[ラムダ](#lambdas)を介してマクロにコードを提供できます。

これらの式置換メカニズムにより、`macro`関数は[他のプログラミング言語で見つかるマクロ](<https://en.wikipedia.org/wiki/Macro_(computer_science)>)と類似しています。ただし、Moveでは他の言語から期待されるよりも制約があります。`macro`関数のパラメータと戻り値は依然として型付けされていますが、これは[`_`型](./../generics#_-type)で部分的に緩和できます。ただし、この制限の利点は、`macro`関数が通常の関数が使用できる場所ならどこでも使用できることで、これは[メソッド構文](./../method-syntax)で特に有用です。

より包括的な[構文マクロ](<https://en.wikipedia.org/wiki/Macro_(computer_science)#Syntactic_macros>)システムが将来追加される可能性があります。

## 構文

`macro`関数は通常の関数と同様の構文を持ちます。ただし、すべての型パラメータ名とすべてのパラメータ名は`$`で始まる必要があります。`_`は単独で使用できますが、プレフィックスとしては使用できず、代わりに`$_`を使用する必要があることに注意してください。

```text
<visibility>? macro fun <identifier><[$type_parameters: constraint],*>([$identifier: type],*): <return_type> <function_body>
```

例えば、以下の`macro`関数はベクターとラムダを取り、ベクターの各要素にラムダを適用して新しいベクターを構築します。

```move
macro fun map<$T, $U>($v: vector<$T>, $f: |$T| -> $U): vector<$U> {
    let mut v = $v;
    v.reverse();
    let mut i = 0;
    let mut result = vector[];
    while (!v.is_empty()) {
        result.push_back($f(v.pop_back()));
        i = i + 1;
    };
    result
}
```

`$`は、パラメータ（型パラメータと値パラメータの両方）が通常の非マクロの対応物とは異なる動作をすることを示すためにあります。型パラメータの場合、任意の型（参照型`&`や`&mut`を含む）でインスタンス化でき、任意の制約を満たします。同様に、パラメータは熱心に評価されるのではなく、代わりに引数式が各使用箇所で置換されます。

## ラムダ

ラムダは`macro`でのみ使用できる新しい型の式です。これらは呼び出し元から`macro`の本体にコードを渡すために使用されます。置換はコンパイル時に行われますが、他の言語の[匿名関数](https://en.wikipedia.org/wiki/Anonymous_function)、[ラムダ](https://en.wikipedia.org/wiki/Lambda_calculus)、または[クロージャ](<https://en.wikipedia.org/wiki/Closure_(computer_programming)>)と同様に使用されます。

As seen in the example above (`$f: |$T| -> $U`), lambda types are defined with the syntax

```text
|<type>,*| (-> <type>)?
```

いくつかの例：

```move
|u64, u64| -> u128 // 2つのu64を取り、u128を返すラムダ
|&mut vector<u8>| -> &mut u8 // &mut vector<u8>を取り、&mut u8を返すラムダ
```

戻り値の型が注釈されていない場合、デフォルトでユニット`()`です。

```move
// 以下は同等です
|&mut vector<u8>, u64|
|&mut vector<u8>, u64| -> ()
```

ラムダ式は、`macro`の呼び出しサイトで以下の構文で定義されます

```text
|(<identifier> (: <type>)?),*| <expression>
|(<identifier> (: <type>)?),*| -> <type> { <expression> }
```

戻り値の型が注釈されている場合、ラムダの本体は`{}`で囲む必要があることに注意してください。

上記で定義された`map`マクロを使用

```move
let v = vector[1, 2, 3];
let doubled: vector<u64> = map!(v, |x| 2 * x);
let bytes: vector<vector<u8>> = map!(v, |x| std::bcs::to_bytes(&x));
```

型注釈付きで

```move
let doubled: vector<u64> = map!(v, |x: u64| 2 * x); // 戻り値の型注釈はオプション
let bytes: vector<vector<u8>> = map!(v, |x: u64| -> vector<u8> { std::bcs::to_bytes(&x) });
```

### キャプチャ

ラムダ式は、ラムダが定義されたスコープ内の変数を参照することもできます。これは
「キャプチャ」と呼ばれることがあります。

```move
let res = foo();
let incremented = map!(vector[1, 2, 3], |x| x + res);
```

可変参照と不変参照を含む任意の変数をキャプチャできます。

See the [Examples](#iterating-over-a-vector) section for more complicated usages.

### 制限

現在、ラムダは`macro`関数の呼び出しで直接使用することしかできません。変数に
バインドすることはできません。例えば、以下のコードはエラーを生成します：

```move
let f = |x| 2 * x;
//      ^^^^^^^^^ エラー！ラムダは'macro'呼び出しで直接使用する必要があります
let doubled: vector<u64> = map!(vector[1, 2, 3], f);
```

## 型付け

通常の関数と同様に、`macro`関数は型付けされています -- パラメータと戻り値の型は
注釈する必要があります。しかし、関数の本体はマクロが展開されるまで型チェックされません。
これは、与えられたマクロのすべての使用が有効であるとは限らないことを意味します。例えば

```move
macro fun add_one<$T>($x: $T): $T {
    $x + 1
}
```

上記のマクロは、`$T`がプリミティブ整数型でない場合、型チェックされません。

これは[メソッド構文](./../method-syntax)と組み合わせて特に有用で、マクロが展開されるまで
関数が解決されません。

```move
macro fun call_foo<$T, $U>($x: $T): &$U {
    $x.foo()
}
```

This macro will only expand successfully if `$T` has a method `foo` that returns a reference `&$U`.
As described in the [hygiene](#hygiene) section, `foo` will be resolved based on the scope where
`call_foo` was defined--not where it was expanded.

### 型パラメータ

型パラメータは、参照型`&`や`&mut`を含む任意の型でインスタンス化できます。また、
[タプル型](./../primitive-types/tuples)でインスタンス化することもできますが、タプルを変数に
バインドできないため、現在この有用性は限られています。

この緩和により、型パラメータの制約が通常発生しない方法で呼び出しサイトで満たされることを
強制します。しかし、型パラメータに必要なすべての制約を追加することを一般的に推奨します。
例えば

```move
public struct NoAbilities()
public struct CopyBox<T: copy> has copy, drop { value: T }
macro fun make_box<$T>($x: $T): CopyBox<$T> {
    CopyBox { value: $x }
}
```

このマクロは、`$T`が`copy`能力を持つ型でインスタンス化された場合にのみ展開されます。

```move
make_box!(1); // Valid!
make_box!(NoAbilities()); // Error! 'NoAbilities' does not have the copy ability
```

`make_box`の推奨宣言は、型パラメータに`copy`制約を追加することです。
これにより、呼び出し元に型が`copy`能力を持たなければならないことを伝えます。

```move
macro fun make_box<$T: copy>($x: $T): CopyBox<$T> {
    CopyBox { value: $x }
}
```

では、推奨が使用しないことであるなら、なぜこの緩和があるのかと合理的に尋ねるかもしれません。
型パラメータの制約は、本体が展開されるまでチェックされないため、すべてのケースで
強制することは単純にできません。以下の例では、`$T`の`copy`制約は署名では必要ありませんが、
本体では必要です。

```move
macro fun read_ref<$T>($r: &$T): $T {
    *$r
}
```

しかし、非常に緩い型署名を持ちたい場合は、代わりに[`_`型](#_-type)を使用することを
推奨します。

### `_`型

通常、[`_`プレースホルダー型](./../generics#_-type)は式で使用され、型引数の部分的な
注釈を可能にします。しかし、`macro`関数では、`_`型を型パラメータの代わりに使用して、
任意の型に対して署名を緩和できます。これにより、「ジェネリック」`macro`関数の宣言の
エルゴノミクスが向上するはずです。

例えば、任意の整数の組み合わせを取り、それらを加算できます。

```move
macro fun add($x: _, $y: _, $z: _): u256 {
    ($x as u256) + ($y as u256) + ($z as u256)
}
```

さらに、`_`型は異なる型で_複数回_インスタンス化できます。例えば

```move
public struct Box<T> has copy, drop, store { value: T }
macro fun create_two($f: |_| -> Box<_>): (Box<u8>, Box<u16>) {
    ($f(0u8), $f(0u16))
}
```

代わりに型パラメータで関数を宣言した場合、型は共通の型に統一される必要がありますが、
この場合は不可能です。

```move
macro fun create_two<$T>($f: |$T| -> Box<$T>): (Box<u8>, Box<u16>) {
    ($f(0u8), $f(0u16))
    //           ^^^^ Error! expected `u8` but found `u16`
}
...
let (a, b) = create_two!(|value| Box { value });
```

In this case, `$T` must be instantiated with a single type, but inference finds that `$T` must be
bound to both `u8` and `u16`.

しかし、`_`型は呼び出し元にとって意味と意図をより少なく伝えるため、トレードオフがあります。
上記の`map`マクロを`$T`と`$U`の代わりに`_`で再宣言することを考えてみてください。

```move
macro fun map($v: vector<_>, $f: |_| -> _): vector<_> {
```

型レベルで`$f`の動作を示すものはもはやありません。呼び出し元はコメントやマクロの本体から
理解を得る必要があります。

## 展開と置換

`macro`の本体はコンパイル時に呼び出しサイトに置換されます。各パラメータは
引数の_式_（値ではなく）で置換されます。ラムダの場合、追加のローカル変数は
`macro`本体のコンテキスト内で値がバインドされることがあります。

非常に簡単な例を取ってみましょう

```move
macro fun apply($f: |u64| -> u64, $x: u64): u64 {
    $f($x)
}
```

呼び出しサイトで

```move
let incremented = apply!(|x| x + 1, 5);
```

これはおおよそ以下のように展開されます：

```move
let incremented = {
    let x = { 5 };
    { x + 1 }
};
```

再び、`x`の値は置換されませんが、式`5`は置換されます。これは、`macro`の本体に応じて、
引数が複数回評価されるか、全く評価されない可能性があることを意味します。

```move
macro fun dup($f: |u64, u64| -> u64, $x: u64): u64 {
    $f($x, $x)
}
```

```move
let sum = dup!(|x, y| x + y, foo());
```

is expanded to

```move
let sum = {
    let x = { foo() };
    let y = { foo() };
    { x + y }
};
```

`foo()`が2回呼び出されることに注意してください。これは`dup`が通常の関数であった場合には
発生しません。

引数をローカル変数にバインドすることで予測可能な評価動作を作成することがしばしば推奨されます。

```move
macro fun dup($f: |u64, u64| -> u64, $x: u64): u64 {
    let a = $x;
    $f(a, a)
}
```

今度は同じ呼び出しサイトが以下のように展開されます：

```move
let sum = {
    let a = { foo() };
    {
        let x = { a };
        let y = { a };
        { x + y }
    }
};
```

### ハイジーン

上記の例では、`dup`マクロには引数`$x`をバインドするために使用されたローカル変数`a`がありました。
変数が代わりに`x`と名付けられた場合、何が起こるでしょうか？ラムダの`x`と競合するでしょうか？

短い答えは、いいえです。`macro`関数は[ハイジーン](https://en.wikipedia.org/wiki/Hygienic_macro)で、
`macro`とラムダの展開が他のスコープから変数を誤ってキャプチャしないことを意味します。

コンパイラは各スコープに一意の番号を関連付けることでこれを行います。`macro`が展開されると、
マクロ本体は独自のスコープを取得します。さらに、引数は各使用で再スコープされます。

`dup`マクロを`a`の代わりに`x`を使用するように修正

```move
macro fun dup($f: |u64, u64| -> u64, $x: u64): u64 {
    let a = $x;
    $f(a, a)
}
```

呼び出しサイトの展開

```move
// let sum = dup!(|x, y| x + y, foo());
let sum = {
    let x#1 = { foo() };
    {
        let x#2 = { x#1 };
        let y#2 = { x#1 };
        { x#2 + y#2 }
    }
};
```

これはコンパイラの内部表現の近似で、この例の簡潔さのためにいくつかの詳細は省略されています。

そして、引数の各使用は再スコープされるため、異なる使用が競合しません。

```move
macro fun apply_twice($f: |u64| -> u64, $x: u64): u64 {
    $f($x) + $f($x)
}
```

```move
let result = apply_twice!(|x| x + 1, { let x = 5; x });
```

Expands to

```move
let result = {
    {
        let x#1 = { let x#2 = { 5 }; x#2 };
        { x#1 + x#1 }
    }
    +
    {
        let x#3 = { let x#4 = { 5 }; x#4 };
        { x#3 + x#3 }
    }
};
```

変数のハイジーンと同様に、[メソッド解決](./../method-syntax)もマクロ定義にスコープされます。
例えば

```move
public struct S { f: u64, g: u64 }

fun f(s: &S): u64 {
    s.f
}
fun g(s: &S): u64 {
    s.g
}

use fun f as foo;
macro fun call_foo($s: &S): u64 {
    let s = $s;
    s.foo()
}
```

この場合、メソッド呼び出し`foo`は常に関数`f`に解決され、`call_foo`が`foo`が`g`などの
異なる関数にバインドされたスコープで使用された場合でもそうです。

```move
fun example(s: &S): u64 {
    use fun g as foo;
    call_foo!(s) // 'g(s)'ではなく'f(s)'に展開される
}
```

このため、未使用の`use fun`宣言は`macro`関数を含むモジュールで警告を受けない可能性があります。

### 制御フロー

変数のハイジーンと同様に、制御フロー構文も常に定義された場所にスコープされ、
展開された場所ではありません。

```move
macro fun maybe_div($x: u64, $y: u64): u64 {
    let x = $x;
    let y = $y;
    if (y == 0) return 0;
    x / y
}
```

呼び出しサイトでは、`return`は常に`macro`本体から返され、呼び出し元からではありません。

```move
let result: vector<u64> = vector[maybe_div!(10, 0)];
```

これは以下のように展開されます：

```move
let result: vector<u64> = vector['a: {
    let x = { 10 };
    let y = { 0 };
    if (y == 0) return 'a 0;
    x / y
}];
```

ここで`return 'a 0`はブロック`'a: { ... }`に返され、呼び出し元の本体には返されません。
詳細については[ラベル付き制御フロー](./../control-flow/labeled-control-flow)のセクションを
参照してください。

同様に、ラムダ内の`return`はラムダから返され、`macro`本体からではなく、外側の関数からも
返されません。

```move
macro fun apply($f: |u64| -> u64, $x: u64): u64 {
    $f($x)
}
```

そして

```move
let result = apply!(|x| { if (x == 0) return 0; x + 1 }, 100);
```

は以下のように展開されます：

```move
let result = {
    let x = { 100 };
    'a: {
        if (x == 0) return 'a 0;
        x + 1
    }
};
```

ラムダから返すことに加えて、ラベルを使用して外側の関数に返すことができます。
`vector::any`マクロでは、ラベル付きの`return`を使用して`macro`全体から早期に返します

```move
public macro fun any<$T>($v: &vector<$T>, $f: |&$T| -> bool): bool {
    let v = $v;
    'any: {
        v.do_ref!(|e| if ($f(e)) return 'any true);
        false
    }
}
```

`return 'any true`は条件が満たされたときに「ループ」から早期に終了します。そうでなければ、
マクロは`false`を「返します」。

### メソッド構文

適用可能な場合、`macro`関数は[メソッド構文](./../method-syntax)を使用して呼び出すことができます。
メソッド構文を使用する場合、引数の評価が変更され、最初の引数（メソッドの「レシーバー」）が
マクロ展開の外で評価されます。この例は人工的ですが、動作を簡潔に示します。

```move
public struct S() has copy, drop;
public fun foo(): S { abort 0 }
public macro fun maybe_s($s: S, $cond: bool): S {
    if ($cond) $s
    else S()
}
```

`foo()`がabortするにもかかわらず、その戻り値の型を使用してメソッド呼び出しを開始できます。

`$cond`が`false`の場合、`$s`は評価されず、通常の非メソッド呼び出しでは、`foo()`の引数は
評価されず、abortしません。以下の例は、`$s`が`foo()`の引数で評価されないことを示しています。

```move
maybe_s!(foo(), false) // abortしません
```

展開された形式を見ると、なぜabortしないのかがより明確になります

```move
if (false) foo()
else S()
```

しかし、メソッド構文を使用する場合、最初の引数はマクロが展開される前に評価されます。
したがって、`$s`の`foo()`の同じ引数が今度は評価され、abortします。

```move
foo().maybe_s!(false) // abortします
```

展開された形式を見ると、これをより明確に確認できます

```move
let tmp = foo(); // aborts
if (false) tmp
else S()
```

概念的には、メソッド呼び出しのレシーバーはマクロが展開される前に一時変数にバインドされ、
これが評価を強制し、したがってabortを引き起こします。

### パラメータの制限

`macro`関数のパラメータは常に式として使用される必要があります。引数が再解釈される可能性がある
状況では使用できません。例えば、以下は許可されません

```move
macro fun no($x: _): _ {
    $x.f
}
```

理由は、引数`$x`が参照でない場合、最初に借用され、これが引数を再解釈する可能性があるためです。
この制限を回避するには、引数をローカル変数にバインドする必要があります。

```move
macro fun yes($x: _): _ {
    let x = $x;
    x.f
}
```

## 例

### 遅延引数: assert_eq

```move
macro fun assert_eq<$T>($left: $T, $right: $T, $code: u64) {
    let left = $left;
    let right = $right;
    if (left != right) {
        std::debug::print(&b"assertion failed.\n left: ");
        std::debug::print(&left);
        std::debug::print(&b"\n does not equal right: ");
        std::debug::print(&right);
        abort $code;
    }
}
```

この場合、アサーションが失敗しない限り、`$code`への引数は評価されません。

```move
assert_eq!(vector[true, false], vector[true, false], 1 / 0); // ゼロ除算は評価されません
```

### 任意の整数平方根

このマクロは、`u256`以外の任意の整数型の整数平方根を計算します。

`$T`は入力の型で、`$bitsize`はその型のビット数です。例えば`u8`は8ビットです。
`$U`は次のより大きな整数型に設定する必要があります。例えば`u8`の場合は`u16`です。

この`macro`では、整数リテラル`1`と`0`の型が注釈されています（例：`(1: $U)`）。
これにより、リテラルの型が各呼び出しで異なることが可能になります。同様に、`as`を
型パラメータ`$T`と`$U`と一緒に使用できます。このマクロは、`$T`と`$U`が整数型で
インスタンス化された場合にのみ正常に展開されます。

```move
macro fun num_sqrt<$T, $U>($x: $T, $bitsize: u8): $T {
    let x = $x;
    let mut bit = (1: $U) << $bitsize;
    let mut res = (0: $U);
    let mut x = x as $U;

    while (bit != 0) {
        if (x >= res + bit) {
            x = x - (res + bit);
            res = (res >> 1) + bit;
        } else {
            res = res >> 1;
        };
        bit = bit >> 2;
    };

    res as $T
}
```

### ベクターの反復

2つの`macro`は、それぞれ不変と可変でベクターを反復します。

```move
macro fun for_imm<$T>($v: &vector<$T>, $f: |&$T|) {
    let v = $v;
    let n = v.length();
    let mut i = 0;
    while (i < n) {
        $f(&v[i]);
        i = i + 1;
    }
}

macro fun for_mut<$T>($v: &mut vector<$T>, $f: |&mut $T|) {
    let v = $v;
    let n = v.length();
    let mut i = 0;
    while (i < n) {
        $f(&mut v[i]);
        i = i + 1;
    }
}
```

使用例

```move
fun imm_examples(v: &vector<u64>) {
    // print all elements
    for_imm!(v, |x| std::debug::print(x));

    // sum all elements
    let mut sum = 0;
    for_imm!(v, |x| sum = sum + x);

    // find the max element
    let mut max = 0;
    for_imm!(v, |x| if (x > max) max = x);
}

fun mut_examples(v: &mut vector<u64>) {
    // increment each element
    for_mut!(v, |x| *x = *x + 1);

    // set each element to the previous value, and the first to last value
    let mut prev = v[v.length() - 1];
    for_mut!(v, |x| {
        let tmp = *x;
        *x = prev;
        prev = tmp;
    });

    // set the max element to 0
    let mut max = &mut 0;
    for_mut!(v, |x| if (*x > *max) max = x);
    *max = 0;
}
```

### 非ループラムダの使用

ラムダはループで使用する必要はなく、条件付きでコードを適用するのにしばしば有用です。

```move
macro fun inspect<$T>($opt: &Option<$T>, $f: |&$T|) {
    let opt = $opt;
    if (opt.is_some()) $f(opt.borrow())
}

macro fun is_some_and<$T>($opt: &Option<$T>, $f: |&$T| -> bool): bool {
    let opt = $opt;
    if (opt.is_some()) $f(opt.borrow())
    else false
}

macro fun map<$T, $U>($opt: Option<$T>, $f: |$T| -> $U): Option<$U> {
    let opt = $opt;
    if (opt.is_some()) {
        option::some($f(opt.destroy_some()))
    } else {
        opt.destroy_none();
        option::none()
    }
}
```

そして使用例

```move
fun examples(opt: Option<u64>) {
    // print the value if it exists
    inspect!(&opt, |x| std::debug::print(x));

    // check if the value is 0
    let is_zero = is_some_and!(&opt, |x| *x == 0);

    // upcast the u64 to a u256
    let str_opt = map!(opt, |x| x as u256);
}
```
