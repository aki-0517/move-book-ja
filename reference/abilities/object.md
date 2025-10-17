---
title: 'Suiオブジェクト | リファレンス'
---

# Suiオブジェクト

Suiでは、`key`は_オブジェクト_を示すために使用されます。オブジェクトはSuiでデータを保存する唯一の方法であり、データがトランザクション間で永続化されることを可能にします。

詳細については、Suiドキュメントを参照してください：

- [オブジェクトモデル](https://docs.sui.io/concepts/object-model)
- [オブジェクトのMoveルール](https://docs.sui.io/concepts/sui-move-concepts#global-unique)
- [オブジェクトの転送](https://docs.sui.io/concepts/transfers)

## オブジェクトルール

オブジェクトは[`key`](../abilities.md#key)アビリティを持つ[`struct`](../structs.md)です。構造体の最初のフィールドは`id: sui::object::UID`である必要があります。この32バイトフィールド（[`address`](../primitive-types/address.md)の強型ラッパー）は、オブジェクトを一意に識別するために使用されます。

`sui::object::UID`は`store`アビリティのみを持ち（`copy`や`drop`は持たない）ため、オブジェクトは`copy`や`drop`を持たないことに注意してください。

## 転送ルール

オブジェクトは`sui::transfer`モジュールで所有権を変更し、転送することができます。モジュール内の多くの関数には「public」と「private」のバリアントがあり、「private」バリアントはオブジェクトの型を定義するモジュール内でのみ呼び出すことができます。「public」バリアントは、オブジェクトが`store`を持つ場合にのみ呼び出すことができます。

例えば、モジュール`my_module`で定義された2つのオブジェクト`A`と`B`がある場合：

```move
module a::my_module;

public struct A has key {
    id: sui::object::UID,
}

public struct B has key, store {
    id: sui::object::UID,
}
```

`A`は`a::my_module`内でのみ`sui::transfer::transfer`を使用して転送できますが、`B`は`sui::transfer::public_transfer`を使用してどこでも転送できます。これらのルールは、Suiのカスタム型システム（バイトコード検証器）ルールによって強制されます。
