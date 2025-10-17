---
title: 'モジュール | リファレンス'
description: ''
---

# モジュール

**モジュール**は、これらの型で動作する関数と共に型を定義するコアプログラム単位です。構造体型はMoveの[ストレージ](./abilities#key)のスキーマを定義し、モジュール関数はそれらの型の値との相互作用のルールを定義します。モジュール自体もストレージに保存されますが、Moveプログラム内からアクセスすることはできません。ブロックチェーン環境では、モジュールは通常「パブリッシュ」と呼ばれるプロセスでチェーン上に保存されます。パブリッシュ後、[`entry`](./functions#entry-modifier)と[`public`](./functions#visibility)関数は、その特定のMoveインスタンスのルールに従って呼び出すことができます。

## 構文

モジュールは以下の構文を持ちます：

```text
module <address>::<identifier> {
    (<use> | <type> | <function> | <constant>)*
}
```

ここで`<address>`は、モジュールのパッケージを指定する有効な[アドレス](./primitive-types/address)です。

例えば：

```move
module 0::test;

use std::debug;

const ONE: u64 = 1;

public struct Example has copy, drop { i: u64 }

public fun print(x: u64) {
    let sum = x + ONE;
    let example = Example { i: sum };
    debug::print(&sum)
}
```

## 名前

`module test_addr::test`部分は、モジュール`test`が[パッケージ設定](./packages)で`test_addr`という名前に割り当てられた数値[アドレス](./primitive-types/address)値の下でパブリッシュされることを指定します。

モジュールは通常、[名前付きアドレス](./primitive-types/address)を使用して宣言する必要があります（数値を直接使用するのではなく）。例えば：

```move
module test_addr::test;

use std::debug;
use test_addr::another_test;

public struct Example has copy, drop { a: address }

public fun print() {
    let example = Example { a: @test_addr };
    debug::print(&example)
}
```

これらの名前付きアドレスは通常、[パッケージ](./packages)の名前と一致します。

名前付きアドレスはソース言語レベルとコンパイル中にのみ存在するため、名前付きアドレスはバイトコードレベルでその値に完全に置換されます。例えば、以下のコードがある場合：

```move
fun example() {
    my_addr::m::foo(@my_addr);
}
```

`my_addr`を`0xC0FFEE`に設定してコンパイルした場合、以下のコードと機能的に同等になります：

```move
fun example() {
    0xC0FFEE::m::foo(@0xC0FFEE);
}
```

ソースレベルでは、これら2つの異なるアクセスは同等ですが、常に名前付きアドレスを使用し、そのアドレスに割り当てられた数値を使用しないことがベストプラクティスです。

モジュール名は`a`から`z`の小文字または`A`から`Z`の大文字で始まることができます。最初の文字の後、モジュール名にはアンダースコア`_`、`a`から`z`の文字、`A`から`Z`の文字、または`0`から`9`の数字を含めることができます。

```move
module a::my_module {}
module a::foo_bar_42 {}
```

通常、モジュール名は小文字で始まります。`my_module`という名前のモジュールは、`my_module.move`という名前のソースファイルに保存する必要があります。

## メンバー

`module`ブロック内のすべてのメンバーは任意の順序で現れることができます。基本的に、モジュールは[`types`](./structs)と[`functions`](./functions)のコレクションです。[`use`](./uses)キーワードは他のモジュールからのメンバーを参照します。[`const`](./constants)キーワードは、モジュールの関数で使用できる定数を定義します。

[`friend`](./friends)構文は、信頼されたモジュールのリストを指定するための非推奨の概念です。この概念は[`public(package)`](./functions#visibility)に置き換えられました

<!-- TODO member access rules -->
