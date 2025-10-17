---
title: 'Uses とエイリアス | リファレンス'
description: ''
---

# Uses とエイリアス

`use`構文を使用して、他のモジュールのメンバーへのエイリアスを作成できます。`use`は、モジュール全体、または特定の式ブロックスコープのいずれかで有効なエイリアスを作成するために使用できます。

## 構文

`use`にはいくつかの異なる構文ケースがあります。最もシンプルなものから始めて、他のモジュールへのエイリアスを作成するために以下があります

```move
use <address>::<module name>;
use <address>::<module name> as <module alias name>;
```

例えば

```move
use std::vector;
use std::option as o;
```

`use std::vector;`は`std::vector`のエイリアス`vector`を導入します。これは、`std::vector`というモジュール名を使用したい場所（この`use`がスコープ内にあると仮定）で、代わりに`vector`を使用できることを意味します。`use std::vector;`は`use std::vector as vector;`と同等です

同様に`use std::option as o;`は、`std::option`の代わりに`o`を使用できるようにします

```move
use std::vector;
use std::option as o;

fun new_vec(): vector<o::Option<u8>> {
    let mut v = vector[];
    vector::push_back(&mut v, o::some(0));
    vector::push_back(&mut v, o::none());
    v
}
```

特定のモジュールメンバー（関数や構造体など）をインポートしたい場合。以下の構文を使用できます。

```move
use <address>::<module name>::<module member>;
use <address>::<module name>::<module member> as <member alias>;
```

For example

```move
use std::vector::push_back;
use std::option::some as s;
```

これにより、完全修飾なしで関数`std::vector::push_back`を使用できます。同様に`std::option::some`は`s`で使用できます。代わりに、それぞれ`push_back`と`s`を使用できます。繰り返しますが、`use std::vector::push_back;`は`use std::vector::push_back as push_back;`と同等です。

```move
use std::vector::push_back;
use std::option::some as s;

fun new_vec(): vector<std::option::Option<u8>> {
    let mut v = vector[];
    vector::push_back(&mut v, s(0));
    vector::push_back(&mut v, std::option::none());
    v
}
```

### 複数のエイリアス

複数のモジュールメンバーに一度にエイリアスを追加したい場合は、以下の構文を使用できます。

```move
use <address>::<module name>::{<module member>, <module member> as <member alias> ... };
```

For example

```move
use std::vector::push_back;
use std::option::{some as s, none as n};

fun new_vec(): vector<std::option::Option<u8>> {
    let mut v = vector[];
    push_back(&mut v, s(0));
    push_back(&mut v, n());
    v
}
```

### Selfエイリアス

モジュールメンバーに加えてモジュール自体にエイリアスを追加する必要がある場合は、`Self`を使用して単一の`use`で行うことができます。`Self`はモジュールを参照する一種のメンバーです。

```move
use std::option::{Self, some, none};
```

明確にするために、以下はすべて同等です：

```move
use std::option;
use std::option as option;
use std::option::Self;
use std::option::Self as option;
use std::option::{Self};
use std::option::{Self as option};
```

### 同じ定義に対する複数のエイリアス

必要に応じて、任意のアイテムに対して好きなだけ多くのエイリアスを持つことができます。

```move
use std::vector::push_back;
use std::option::{Option, some, none};

fun new_vec(): vector<Option<u8>> {
    let mut v = vector[];
    push_back(&mut v, some(0));
    push_back(&mut v, none());
    v
}
```

### ネストしたインポート

Moveでは、同じ`use`宣言で複数の名前をインポートすることもできます。これにより、提供されたすべての名前がスコープに取り込まれます：

```move
use std::{
    vector::{Self as vec, push_back},
    string::{String, Self as str}
};

fun example(s: &mut String) {
    let mut v = vec::empty();
    push_back(&mut v, 0);
    push_back(&mut v, 10);
    str::append_utf8(s, v);
}
```

## `module`内

`module`内では、宣言の順序に関係なく、すべての`use`宣言が使用可能です。

```move
module a::example;

use std::vector;

fun new_vec(): vector<Option<u8>> {
    let mut v = vector[];
    vector::push_back(&mut v, 0);
    vector::push_back(&mut v, 10);
    v
}

use std::option::{Option, some, none};
```

モジュール内で`use`によって宣言されたエイリアスは、そのモジュール内で使用可能です。

さらに、導入されたエイリアスは他のモジュールメンバーと競合することはできません。詳細は[一意性](#uniqueness)を参照してください。

## 式内

任意の式ブロックの先頭に`use`宣言を追加できます。

```move
module a::example;

fun new_vec(): vector<Option<u8>> {
    use std::vector::push_back;
    use std::option::{Option, some, none};

    let mut v = vector[];
    push_back(&mut v, some(0));
    push_back(&mut v, none());
    v
}
```

`let`と同様に、式ブロック内で`use`によって導入されたエイリアスは、そのブロックの終わりで削除されます。

```move
module a::example;

fun new_vec(): vector<Option<u8>> {
    let result = {
        use std::vector::push_back;
        use std::option::{Option, some, none};

        let mut v = vector[];
        push_back(&mut v, some(0));
        push_back(&mut v, none());
        v
    };
    result
}
```

ブロック終了後にエイリアスを使用しようとするとエラーになります。

```move
fun new_vec(): vector<Option<u8>> {
    let mut result = {
        use std::vector::push_back;
        use std::option::{Option, some, none};

        let mut v = vector[];
        push_back(&mut v, some(0));
        v
    };
    push_back(&mut result, std::option::none());
    // ^^^^^^ ERROR! unbound function 'push_back'
    result
}
```

任意の`use`はブロックの最初の項目である必要があります。`use`が任意の式や`let`の後に来る場合、解析エラーになります。

```move
{
    let mut v = vector[];
    use std::vector; // ERROR!
}
```

これにより、多くの状況でインポートブロックを短縮できます。これらのインポートは、前のものと同様に、以下のセクションで説明する命名規則と一意性規則の対象となることに注意してください。

## 命名規則

エイリアスは他のモジュールメンバーと同じ規則に従う必要があります。これは、構造体（および定数）のエイリアスが`A`から`Z`で始まる必要があることを意味します。

```move
module a::data {
    public struct S {}
    const FLAG: bool = false;
    public fun foo() {}
}
module a::example {
    use a::data::{
        S as s, // ERROR!
        FLAG as fLAG, // ERROR!
        foo as FOO,  // valid
        foo as bar, // valid
    };
}
```

## 一意性

与えられたスコープ内では、`use`宣言によって導入されたすべてのエイリアスは一意である必要があります。

モジュールの場合、これは`use`によって導入されたエイリアスが重複できないことを意味します。

```move
module a::example;

use std::option::{none as foo, some as foo}; // ERROR!
//                                     ^^^ duplicate 'foo'

use std::option::none as bar;

use std::option::some as bar; // ERROR!
//                       ^^^ duplicate 'bar'
```

また、モジュールの他のメンバーと重複することはできません。

```move
module a::data {
    public struct S {}
}

module example {
    use a::data::S;

    public struct S { value: u64 } // ERROR!
    //            ^ conflicts with alias 'S' above
}
```

式ブロック内では、互いに重複することはできませんが、外側のスコープの他のエイリアスや名前を[シャドウ](#shadowing)できます。

## シャドウイング

式ブロック内の`use`エイリアスは、外側のスコープの名前（モジュールメンバーまたはエイリアス）をシャドウできます。ローカルのシャドウイングと同様に、シャドウイングは式ブロックの終わりで終了します。

```move
module a::example;

public struct WrappedVector { vec: vector<u64> }

public fun empty(): WrappedVector {
    WrappedVector { vec: std::vector::empty() }
}

public fun push_back(v: &mut WrappedVector, value: u64) {
    std::vector::push_back(&mut v.vec, value);
}

fun example1(): WrappedVector {
    use std::vector::push_back;
    // 'push_back' now refers to std::vector::push_back
    let mut vec = vector[];
    push_back(&mut vec, 0);
    push_back(&mut vec, 1);
    push_back(&mut vec, 10);
    WrappedVector { vec }
}

fun example2(): WrappedVector {
    let vec = {
        use std::vector::push_back;
        // 'push_back' now refers to std::vector::push_back

        let mut v = vector[];
        push_back(&mut v, 0);
        push_back(&mut v, 1);
        v
    };
    // 'push_back' now refers to Self::push_back
    let mut res = WrappedVector { vec };
    push_back(&mut res, 10);
    res
}
```

## 未使用のUseまたはエイリアス

未使用の`use`は警告を発生させます。

```move
module a::example;

use std::option::{some, none}; // Warning!
//                      ^^^^ unused alias 'none'

public fun example(): std::option::Option<u8> {
    some(0)
}
```
