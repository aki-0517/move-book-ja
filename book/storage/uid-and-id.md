# UIDとID

`UID`型の使用は、[`key`](./key-ability.md)アビリティを持つすべての型に対してSui Verifierによって要求されます。ここでは`UID`とその使用法について詳しく説明します。

## 定義

`UID`型は`sui::object`モジュールで定義されており、`ID`のラッパーです。`ID`は`address`型をラップします。SuiのUIDは一意であることが保証され、オブジェクトが削除された後に再利用されることはありません。

```move
module sui::object;

/// UIDはオブジェクトの一意の識別子
public struct UID has store {
    id: ID
}

/// IDはアドレスのラッパー
public struct ID has store, drop {
    bytes: address
}
```

## 新しいUIDの生成

- `UID`は`tx_hash`と各新しいUIDに対してインクリメントされる`index`から_導出_されます。
- `derive_id`関数は`sui::tx_context`モジュールに実装されており、そのため[TxContext](./../programmability/transaction-context.md)が`UID`生成に必要です。
- Sui Verifierは、同じ関数で作成されなかったUIDの使用を許可しません。これにより、UIDが事前生成されたり、オブジェクトがアンパックされた後に再利用されることを防ぎます。

新しいUIDは`object::new`関数で作成されます。これは`TxContext`への可変参照を取り、新しい`UID`を返します。

```move
public fun uid(ctx: &mut TxContext) {
  let id = object::new(ctx); // TxContextから新しいUIDを作成。
  id.delete(); // UIDを削除。
}
```

`UID`はオブジェクトの表現として機能し、オブジェクトの動作と機能を定義することを可能にします。主要な機能の一つ - [動的フィールド](./../programmability/dynamic-fields) - は、`UID`型が明示的であるために可能です。さらに、他のオブジェクトに送信されたオブジェクトを受信することを可能にします。この機能は[Transfer to Object (TTO)](./transfer-to-object.md)と呼ばれ、この章の後半で説明します。

## UID導出

Suiは_導出キー_を使用して他のUIDからUIDを導出することを可能にします。この機能は[`sui::derived_object`][derived-object]モジュールに実装されており、オフチェーン発見を容易にするための予測可能で決定論的な`UID`の生成を可能にします。各親+キーのペアのUIDは一度だけ生成できます！

```move
use sui::derived_object;

/// 何らかの中央アプリケーションオブジェクト。
public struct Base has key { id: UID }

/// 導出オブジェクト。
public struct Derived has key { id: UID }

/// `address`を`key`として使用して新しい導出オブジェクトを作成し、共有。
public fun derive(base: &mut Base, key: address) {
    let id = derived_object::claim(&mut base.id, key);
    transfer::share_object(Derived { id })
}
```

導出アドレスは、親オブジェクトのIDを保存し、導出関数を使用して導出IDを取得するだけで十分であるため、オフチェーンインデクサーの負荷を軽減します。ID導出関数はほとんどのSDKの一部であり、Moveにも存在します：

```move
module sui::derived_object;

/// UIDが`parent`で`key`と導出されたかどうかをチェック。
public fun exists<K: copy + drop + store>(parent: &UID, key: K): bool;

/// UIDが作成されたかどうかに関係なく、UIDの内部`address`を導出。
public fun derive_address<K: copy + drop + store>(parent: ID, key: K): address;
```

同じ導出機能は[動的フィールド](./../programmability/dynamic-fields.md)のUID生成にも使用されます。

## UIDライフサイクル

`UID`型は`object::new`関数で作成され、`object::delete`関数で削除されます。`object::delete`はUIDを_値で_消費するため、オブジェクト[がアンパックされた](./../move-basics/struct.md#unpacking-a-struct)後にのみオブジェクトのUIDを削除することが可能です。

```move
public struct Character has key { id: UID }

public fun character(ctx: &mut TxContext) {
    // `Character`オブジェクトをインスタンス化。
    let char = Character { id: object::new(ctx) };

    // オブジェクトをアンパックしてUIDを取得。
    let Character { id } = char;

    // UIDを削除。
    id.delete();
}
```

## UIDの保持

`UID`はオブジェクト構造体がアンパックされた直後に削除する必要はありません。時には[動的フィールド](./../programmability/dynamic-fields)を保持しているか、[Transfer To Object](./transfer-to-object.md)を通じて転送されたオブジェクトを保持している場合があります。そのような場合、UIDは保持され、別のオブジェクトに保存されることがあります。

## 削除の証明

オブジェクトのUIDを返す能力は、_削除の証明_と呼ばれるパターンで活用されることがあります。これはまれに使用される技術ですが、一部のケースで有用な場合があります。例えば、作成者やアプリケーションが削除されたIDを何らかの報酬と交換することで、オブジェクトの削除を奨励する場合があります。

フレームワーク開発では、この方法を使用してオブジェクトを「取得」することに関する特定の制限を無視/バイパスすることができます。Kioskのように転送に特定のロジックを強制するコンテナがある場合、削除の証明を提供することでチェックをスキップする特別なシナリオがあるかもしれません。

これは探求と研究のためのオープンなトピックの一つであり、様々な方法で使用される可能性があります。

## ID

`UID`について話すとき、`ID`型についても言及すべきです。これは`address`型のラッパーであり、アドレスポインタを表現するために使用されます。通常、`ID`はオブジェクトを指すために使用されますが、制限はなく、`ID`が既存のオブジェクトを指すという保証もありません。

> IDは[トランザクションブロック](./../concepts/what-is-a-transaction)でトランザクション引数として受信できます。または、`to_id()`関数を使用して`address`値からIDを作成できます。

```move
public fun conversion_methods(ctx: &mut TxContext) {
    let uid: UID = object::new(ctx);
    let id: ID = uid.to_inner();

    let addr_from_uid: address = uid.to_address();
    let addr_from_id: address = id.to_address();

    uid.delete();
}
```

この例は異なる変換方法を示しています：`UID.to_inner`は基盤となる`ID`のコピーを作成し、`UID.to_address`は内部アドレスを返します。もう一つのよく使われる方法`ID.to_address`は`ID`型から内部値をコピーします。

## 新しいオブジェクトアドレス

[`TxContext`](./../programmability/transaction-context.md)は、一意のアドレスと`ID`を作成するために活用できる`fresh_object_address`関数を提供します - これは、ユーザーのアクションに一意の識別子を割り当てるアプリケーション（例えば、マーケットプレイスのorder_id）で有用な場合があります。

## リンク

- [`sui::object`][object]モジュールドキュメント
- [`sui::derived_object`][derived-object]モジュールドキュメント
- Suiドキュメントの[Derived Objects](https://docs.sui.io/concepts/sui-move-concepts/derived-objects)

[object]: https://docs.sui.io/references/framework/sui_sui/object
[derived-object]: https://docs.sui.io/references/framework/sui_sui/derived_object