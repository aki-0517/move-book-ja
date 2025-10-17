# アビリティ: Key

[基本構文][basic-syntax]の章では、4つのアビリティのうち2つ、[Drop][drop-ability]と[Copy][copy-ability]について既に説明しました。これらはスコープ内での値の動作に影響し、ストレージと直接関係ありません。今度は`key`アビリティについて説明します。これは構造体を_ストア_できるようにします。

歴史的に、`key`アビリティは型を_ストレージ内のキー_としてマークするために作成されました。`key`アビリティを持つ型は、グローバルストレージのトップレベルに保存でき、アカウントやアドレスによって_所有_されることができました。[オブジェクトモデル][object-model]の導入により、`key`アビリティは_オブジェクト_の定義アビリティとなりました。

> 本書の後半では、`key`アビリティを持つ任意の構造体をオブジェクトと呼びます。

## オブジェクトの定義

`key`アビリティを持つ構造体は_オブジェクト_とみなされ、ストレージ関数で使用できます。Sui Verifierは、構造体の最初のフィールドが`id`という名前で、`UID`型を持つことを要求します。さらに、すべてのフィールドが`store`アビリティを持つことを要求します。これについては[次のページ][store-ability]で詳しく説明します。

```move
/// `User`オブジェクトの定義。
public struct User has key {
    id: UID, // Suiバイトコード検証器によって要求される
    name: String, // フィールド型は`store`を持つ必要がある
}

/// `User`型の新しいインスタンスを作成。
/// 特別な構造体`TxContext`を使用してユニークID（UID）を導出。
public fun new(name: String, ctx: &mut TxContext): User {
    User {
        id: object::new(ctx), // 新しいUIDを作成
        name,
    }
}
```

## `copy`と`drop`との関係

`UID`は[`drop`][drop-ability]や[`copy`][copy-ability]アビリティを持たない型です。`key`アビリティを持つ任意の型のフィールドとして必要であるため、これは`key`を持つ型が`drop`や`copy`を持つことができないことを意味します。

この特性は[アビリティ制約][generics]で活用できます：`drop`や`copy`を要求することは自動的に`key`を除外し、逆に`key`を要求することは`drop`や`copy`を持つ型を除外します。

## `key`アビリティを持つ型

`key`を持つ型の`UID`要件により、Moveのネイティブ型のいずれも`key`アビリティを持つことができず、[Standard Library][standard-library]の型も同様です。`key`アビリティは、一部の[Sui Framework][sui-framework]型とカスタム型にのみ存在します。

## まとめ

- `key`アビリティはオブジェクトを定義する
- オブジェクトの最初のフィールドは`UID`型の`id`でなければならない
- `key`型のフィールドは[`store`][store-ability]アビリティを持たなければならない
- オブジェクトは[`drop`][drop-ability]や[`copy`][copy-ability]を持つことができない

## 次のステップ

`key`アビリティはMoveでオブジェクトを定義し、フィールドに`store`を強制します。次のセクションでは[ストレージ操作](./storage-functions.md)の仕組みを説明するために、[`store`アビリティ](./store-ability.md)について説明します。

## さらなる学習

- Moveリファレンスの[型アビリティ](./../../reference/abilities)。

[drop-ability]: ./../move-basics/drop-ability
[copy-ability]: ./../move-basics/copy-ability
[store-ability]: ./store-ability.md
[generics]: ./../move-basics/generics#constraints-on-type-parameters
[sui-framework]: ./../programmability/sui-framework
[standard-library]: ./../move-basics/standard-library
[object-model]: ./../object
[basic-syntax]: ./../move-basics