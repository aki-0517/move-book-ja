# Address型

<!--

Chapter: Basic Syntax
Goal: Introduce the address type
Notes:
    - a special type
    - named addresses via the Move.toml
    - address literals
    - 0x2 is 0x0000000...02

Links:
    - address concept
    - transaction context
    - Move.toml
    - your first move

 -->

Moveは、アドレスを表現するために[address](./../concepts/address)という特別な型を使用します。これはブロックチェーン上の任意のアドレスを表現できる32バイトの値です。アドレスは2つの形式で記述できます：0xで始まる16進数アドレスと名前付きアドレスです。

```move file=packages/samples/sources/move-basics/address.move anchor=address_literal

```

アドレスリテラルは`@`記号で始まり、その後に16進数または識別子が続きます。16進数は32バイト値として解釈されます。識別子は[Move.toml](./../concepts/manifest)ファイルで検索され、コンパイラによって対応するアドレスに置き換えられます。識別子がMove.tomlファイルで見つからない場合、コンパイラはエラーを投げます。

## 変換

Sui Frameworkはアドレスを操作するためのヘルパー関数のセットを提供します。address型は32バイト値であるため、`u256`型に変換したり、その逆も可能です。また、`vector<u8>`型との相互変換も可能です。

例：アドレスを`u256`型に変換して戻す。

```move file=packages/samples/sources/move-basics/address.move anchor=to_u256

```

例：アドレスを`vector<u8>`型に変換して戻す。

```move file=packages/samples/sources/move-basics/address.move anchor=to_bytes

```

例：アドレスを文字列に変換する。

```move file=packages/samples/sources/move-basics/address.move anchor=to_string

```

## さらなる読み物

- Move Referenceの[Address](./../../reference/primitive-types/address)
- [sui::address](https://docs.sui.io/references/framework/sui/address)モジュールドキュメント
