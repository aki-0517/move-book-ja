# オブジェクトとして受信

[アドレス所有](./storage-functions.md#transfer)オブジェクト状態は、2つのタイプの所有者をサポートします：アカウントと別のオブジェクトです。オブジェクトが別のオブジェクトに転送された場合、Suiは所有者の[`UID`][uid]を通じてこのオブジェクトを_受信_する方法を提供します。

> この機能は_"Transfer to Object"_またはTTOとしても知られています。

## 定義

受信機能は[`sui::transfer`][transfer]モジュールに実装されています。これは、特別なトランザクション引数を通じてインスタンス化される特別な型`Receiving`と、親の[`UID`][uid]を取る`receive`関数で構成されています。

> 現在、`transfer::receive`の`T`は[内部制約][internal]の対象です。`receive`のパブリック版は`public_receive`と呼ばれ、他の[ストレージ関数][storage-funs]と同様に`T`が[`store`][store]を持つことを要求します。

```move
module sui::transfer;

// `Receiving`引数の周りの一時的なラッパー。トランザクションブロックで
// 特別な入力として提供される。
// 注意：この型を使用するには明示的にインポートする必要がある！
public struct Receiving<phantom T: key> has drop {
    id: ID,
    version: u64,
}

/// 特別な型`Receiving`を通じて親`UID`から`T`を受信。
public fun receive<T: key>(parent: &mut UID, to_receive: Receiving<T>): T;
```

`UID`型要件により、受信はアクセスを提供しない、または特別な受信実装を提供しない任意のオブジェクトで実行することはできません。この機能は注意深く、制御された設定で使用されるべきです。

## 例

_転送_と_受信_の例として、`PostOffice`がポストボックスを登録し、アカウントのポストボックスに送信することを可能にする例を考えてみましょう。

```move file=packages/samples/sources/storage/transfer-to-object.move anchor=main

```

## 使用ケース

オブジェクトへの転送は、オブジェクトが他のオブジェクトの所有者として機能することを可能にする強力な機能です。これを使用する理由の一つは、受信時に実行される追加の認証です。例えば、上記の例の`PostOffice`は受信料を請求することができます。

- トランザクションでそれらを参照することなく、複数のオブジェクトへの転送の並列実行を可能にする；
- 親オブジェクトも転送でき、コンテナとして機能する；
- ユーザーがアカウントをアクティベートした後にのみアセットを取得するPostBoxのようなアプリケーション；
- オブジェクトがアカウントを模倣するアカウント抽象化のようなアプリケーション。

## リンク

- Suiドキュメントの[Transfer to Object](https://docs.sui.io/concepts/transfers/transfer-to-object)
- [`sui::transfer`][transfer]モジュールドキュメント

[transfer]: https://docs.sui.io/references/framework/sui_sui/transfer
[key]: ./key-ability.md
[store]: ./store-ability.md
[uid]: ./uid-and-id.md
[internal]: ./internal-constraint.md