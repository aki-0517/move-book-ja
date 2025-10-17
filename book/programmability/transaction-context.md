# トランザクションコンテキスト

すべてのトランザクションには実行コンテキストがあります。コンテキストは、実行中にプログラムに
利用可能な事前定義された変数のセットです。例えば、すべてのトランザクションには送信者アドレスがあり、
トランザクションコンテキストには送信者アドレスを保持する変数が含まれています。

トランザクションコンテキストは、`TxContext`構造体を通じてプログラムに利用可能です。
この構造体は[`sui::tx_context`][tx-context-framework]モジュールで定義され、
以下のフィールドを含みます：

[tx-context-framework]: https://docs.sui.io/references/framework/sui/tx_context

```move
module sui::tx_context;

/// 現在実行されているトランザクションに関する情報。
/// これはトランザクションによって構築することはできません - VMによって作成され、
/// `&mut TxContext`としてトランザクションのエントリーポイントに渡される特権オブジェクトです。
public struct TxContext has drop {
    /// 現在のトランザクションに署名したユーザーのアドレス
    sender: address,
    /// 現在のトランザクションのハッシュ
    tx_hash: vector<u8>,
    /// 現在のエポック番号
    epoch: u64,
    /// エポックが開始されたタイムスタンプ
    epoch_timestamp_ms: u64,
    /// このトランザクションの実行中に作成された新しいIDの数を記録するカウンター
    /// トランザクションの開始時は常に0
    ids_created: u64
}
```

トランザクションコンテキストは手動で構築したり直接変更したりすることはできません。
システムによって作成され、トランザクション内で参照として関数に渡されます。
[Transaction](./../concepts/what-is-a-transaction)で呼び出される任意の関数はコンテキストに
アクセスでき、ネストされた呼び出しに渡すことができます。

> `TxContext`は関数シグネチャの最後の引数である必要があります。

## トランザクションコンテキストの読み取り

`ids_created`を除いて、`TxContext`のすべてのフィールドにはゲッターがあります。
ゲッターは`sui::tx_context`モジュールで定義され、プログラムに利用可能です。
ゲッターはコンテキストを変更しないため、`&mut`を必要としません。

```move file=packages/samples/sources/programmability/transaction-context.move anchor=reading

```

## 可変性

`TxContext`は、システム内で新しいオブジェクト（または単に`UID`）を作成するために必要です。
新しいUIDはトランザクションダイジェストから派生され、ダイジェストを一意にするために、
変化するパラメータが必要です。Suiはそのために`ids_created`フィールドを使用します。
新しいUIDが作成されるたびに、`ids_created`フィールドが1つ増加します。
このようにして、ダイジェストは常に一意になります。

内部的には、`derive_id`関数として表現されます：

```move
native fun derive_id(tx_hash: vector<u8>, ids_created: u64): address;
```

## 一意のアドレスの生成

基盤となる`derive_id`関数は、プログラムで一意のアドレスを生成するためにも
利用できます。関数自体は公開されていませんが、`sui::tx_context`モジュールで
`fresh_object_address`ラッパー関数が利用可能です。プログラムで一意の識別子を
生成する必要がある場合に役立つかもしれません。

```move
module sui::tx_context;

/// 使用されていない`address`を作成します。これはオブジェクトアドレスであるため、
/// ユーザーのアドレスとして発生することはありません。
/// 言い換えれば、生成されたアドレスはグローバルに一意のオブジェクトIDです。
public fun fresh_object_address(ctx: &mut TxContext): address {
    let ids_created = ctx.ids_created;
    let id = derive_id(*&ctx.tx_hash, ids_created);
    ctx.ids_created = ids_created + 1;
    id
}
```
