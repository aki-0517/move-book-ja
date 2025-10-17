---
title: 'Address | リファレンス'
description: ''
---

# Address

`address`はMoveに組み込まれた型で、ストレージ内の位置（時々アカウントと呼ばれる）を表すために使用されます。`address`値は256ビット（32バイト）の識別子です。Moveはアドレスを使用して[モジュール](./../modules)のパッケージを区別し、各パッケージは独自のアドレスとモジュールを持ちます。Moveの特定のデプロイでは、[ストレージ](./../abilities#key)操作に`address`値を使用する場合もあります。

> Suiでは、`address`は「アカウント」を表すために使用され、また強型ラッパー（`sui::object::UID`と`sui::object::ID`）を介してオブジェクトも表します。

`address`は内部的には256ビット整数であるにもかかわらず、Moveアドレスは意図的に不透明です――整数から作成できず、算術演算をサポートせず、変更できません。Moveの特定のデプロイでは、これらの操作の一部を有効にする`native`関数がある場合があります（例えば、バイト`vector<u8>`から`address`を作成する）が、これらはMove言語自体の一部ではありません。

ランタイムアドレス値（`address`型の値）は存在しますが、ランタイム時にモジュールにアクセスするために使用することは_できません_。

## アドレスとその構文

アドレスには2つの種類があります：名前付きまたは数値です。名前付きアドレスの構文はMoveの任意の名前付き識別子と同じルールに従います。数値アドレスの構文は16進エンコード値に制限されず、有効な[`u256`数値](./integers)はすべてアドレス値として使用できます。例えば、`42`、`0xCAFE`、`10_000`はすべて有効な数値アドレスリテラルです。

アドレスが式コンテキストで使用されているかどうかを区別するために、アドレスを使用する際の構文は使用されるコンテキストによって異なります：

- アドレスが式として使用される場合、アドレスの前に`@`文字を付ける必要があります。つまり、[`@<numerical_value>`](./integers)または`@<named_address_identifier>`です。
- 式コンテキスト外では、アドレスは前置の`@`文字なしで書くことができます。つまり、[`<numerical_value>`](./integers)または`<named_address_identifier>`です。

一般的に、`@`をアドレスを名前空間アイテムから式アイテムに変換する演算子と考えることができます。

## Named Addresses

Named addresses are a feature that allow identifiers to be used in place of numerical values in any
spot where addresses are used, and not just at the value level. Named addresses are declared and
bound as top level elements (outside of modules and scripts) in Move packages, or passed as
arguments to the Move compiler.

Named addresses only exist at the source language level and will be fully substituted for their
value at the bytecode level. Because of this, modules and module members should be accessed through
the module's named address and not through the numerical value assigned to the named address during
compilation. So while `use my_addr::foo` is equivalent to `use 0x2::foo` (if `my_addr` is assigned
`0x2`), it is a best practice to always use the `my_addr` name.

### Examples

```move
// shorthand for
// 0x0000000000000000000000000000000000000000000000000000000000000001
let a1: address = @0x1;
// shorthand for
// 0x0000000000000000000000000000000000000000000000000000000000000042
let a2: address = @0x42;
// shorthand for
// 0x00000000000000000000000000000000000000000000000000000000DEADBEEF
let a3: address = @0xDEADBEEF;
// shorthand for
// 0x000000000000000000000000000000000000000000000000000000000000000A
let a4: address = @0x0000000000000000000000000000000A;
// Assigns `a5` the value of the named address `std`
let a5: address = @std;
// Any valid numerical value can be used as an address
let a6: address = @66;
let a7: address = @42_000;

module 66::some_module {   // Not in expression context, so no @ needed
    use 0x1::other_module; // Not in expression context so no @ needed
    use std::vector;       // Can use a named address as a namespace item
    ...
}

module std::other_module {  // Can use a named address when declaring a module
    ...
}
```
