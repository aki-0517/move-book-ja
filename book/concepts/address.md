# アドレス（Address）

<!--

Chapter: Concepts
Goal: explain locations and addresses
Notes:
    - don't talk about the type
    - packages, accounts and objects are identified by addresses
    - addresses are 32 bytes long
    - addresses are unique
    - represented as hex strings (64 characters) prefixed with 0x
    - addresses are case insensitive

Links:
    - address type


- mention what an address is, because it identifies a package
    - address is used for packages, objects, and accounts
    - address is a 32-byte value
    - address is written in hexadecimal notation
    - don't describe the type yet
    - focus on the concept of address on blockchain and on Sui in particular

 -->

アドレスは、ブロックチェーン上の場所を示す一意の識別子です。
[パッケージ](./packages)、[アカウント](./what-is-an-account)、および[オブジェクト](./../object/object-model)を識別するために使用されます。
アドレスは32バイトの固定サイズを持ち、通常は`0x`で始まる16進数文字列として表現されます。アドレスは大文字小文字を区別しません。

```move
0xe51ff5cd221a81c3d6e22b9e670ddf99004d71de4f769b0312b68c7c4872e2f1
```

上記のアドレスは有効なアドレスの例です。64文字（32バイト）の長さで、`0x`で始まっています。

Suiには、標準パッケージやオブジェクトを識別するために使用される予約アドレスもあります。予約アドレスは通常、覚えやすく入力しやすい簡単な値です。例えば、標準ライブラリのアドレスは`0x1`です。32バイトより短いアドレスは、左側にゼロが埋め込まれます。

```move
0x1 = 0x0000000000000000000000000000000000000000000000000000000000000001
```

予約アドレスの例を以下に示します：

- `0x1` - Sui標準ライブラリのアドレス（エイリアス`std`）
- `0x2` - Suiフレームワークのアドレス（エイリアス`sui`）
- `0x6` - システムの`Clock`オブジェクトのアドレス

> すべての予約アドレスは[付録B](../appendix/reserved-addresses)で確認できます。

## 関連資料

- Move の[アドレス型](../move-basics/address)
- [sui::address モジュール](https://docs.sui.io/references/framework/sui/address)
