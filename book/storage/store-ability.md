# アビリティ: Store

[`key`アビリティ][key-ability]はすべてのフィールドが`store`を持つことを要求し、これが`store`アビリティの意味を定義しています：オブジェクトのフィールドとして機能する能力です。[`copy`][copy-ability]や[`drop`][drop-ability]を持つが`store`を持たない構造体は、決して_ストア_されることはありません。`key`を持つが`store`を持たない型は、他のオブジェクトにラップ（フィールドとして使用）されることができず、常にトップレベルに留まるように制限されます。

## 定義

`store`アビリティは、型が`key`アビリティを持つ構造体のフィールドとして使用されることを可能にします。

```move
// hidden-block-start
use std::string::String;

// hidden-block-end
/// `store`を持つ追加のメタデータ；すべてのフィールドも`store`を持つ必要がある！
public struct Metadata has store {
    bio: String,
}

/// 単一のユーザーレコード用のオブジェクト。
public struct User has key {
    id: UID,
    name: String,       // Stringは`store`を持つ
    age: u8,            // すべての整数は`store`を持つ
    metadata: Metadata, // `store`アビリティを持つ別の型
}
```

## `copy`と`drop`との関係

3つの非`key`アビリティは任意の組み合わせで使用できます。

## `key`との関係

`store`アビリティを持つオブジェクトは、他のオブジェクトに_ストア_できます。

> 言語や検証器の機能ではありませんが、`store`は構造体の_パブリック_修飾子として機能し、[内部制約](./internal-constraint.md)を持たない_パブリック_[転送関数](./storage-functions.md)の呼び出しを可能にします。

## `store`アビリティを持つ型

Moveのすべてのネイティブ型（参照を除く）は`store`アビリティを持ちます。これには以下が含まれます：

- [bool](./../move-basics/primitive-types.md#booleans)
- [符号なし整数](./../move-basics/primitive-types.md#integer-types)
- [vector](./../move-basics/vector.md)
- [address](./../move-basics/address.md)

標準ライブラリで定義されたすべての型も`store`アビリティを持ちます。これには以下が含まれます：

- [Option](./../move-basics/option.md)
- [String](./../move-basics/string.md)と[ASCII String](./../move-basics/string.md)
- [TypeName](./../move-basics/type-reflection.md)

## さらなる学習

- Moveリファレンスの[型アビリティ](./../../reference/abilities)。

[key-ability]: ./key-ability.md
[drop-ability]: ./../move-basics/drop-ability.md
[copy-ability]: ./../move-basics/copy-ability.md