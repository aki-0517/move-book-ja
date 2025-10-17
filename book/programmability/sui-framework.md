# Sui Framework

Sui Frameworkは[Package Manifest](./../concepts/manifest)でデフォルトの依存関係セットです。
[Standard Library](./../move-basics/standard-library)に依存し、ストレージとの相互作用や
Sui固有のネイティブ型とモジュールを含む、Sui固有の機能を提供します。

_便宜上、Sui Frameworkのモジュールを複数のカテゴリにグループ化しました。しかし、
それらは同じフレームワークの一部です。_

## コア

<!-- Custom CSS addition in the theme/custom.css  -->
<div class="modules-table">

| Module                                                                                         | Description                                                             | Chapter                                                     |
| ---------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------- | ----------------------------------------------------------- |
| [sui::address](https://docs.sui.io/references/framework/sui/address)                           | Adds conversion methods to the [address type](./../move-basics/address) | [Address](./../move-basics/address)                         |
| [sui::transfer](https://docs.sui.io/references/framework/sui/transfer)                         | Implements the storage operations for Objects                           | [Storage Functions](./../storage/storage-functions.md)      |
| [sui::tx_context](https://docs.sui.io/references/framework/sui/tx_context)                     | Contains the `TxContext` struct and methods to read it                  | [Transaction Context](./transaction-context)                |
| [sui::object](https://docs.sui.io/references/framework/sui/object)                             | Defines the `UID` and `ID` type, required for creating objects          | [UID and ID](./../storage/uid-and-id.md)                    |
| [sui::derived_object](https://docs.sui.io/references/framework/sui/derived_object)             | Allows `UID` generation through key derivation                          | [UID Derivation](./../storage/uid-and-id.md#uid-derivation) |
| [sui::clock](https://docs.sui.io/references/framework/sui/clock)                               | Defines the `Clock` type and its methods                                | [Epoch and Time](./epoch-and-time)                          |
| [sui::dynamic_field](https://docs.sui.io/references/framework/sui/dynamic_field)               | Implements methods to add, use and remove dynamic fields                | [Dynamic Fields](./dynamic-fields)                          |
| [sui::dynamic_object_field](https://docs.sui.io/references/framework/sui/dynamic_object_field) | Implements methods to add, use and remove dynamic object fields         | [Dynamic Object Fields](./dynamic-object-fields)            |
| [sui::event](https://docs.sui.io/references/framework/sui/event)                               | Allows emitting events for off-chain listeners                          | [Events](./events)                                          |
| [sui::package](https://docs.sui.io/references/framework/sui/package)                           | Defines the `Publisher` type and package upgrade methods                | [Publisher](./publisher), Package Upgrades                  |
| [sui::display](https://docs.sui.io/references/framework/sui/display)                           | Implements the `Display` object and ways to create and update it        | [Display](./display)                                        |

</div>

## コレクション

<div class="modules-table">

| Module                                                                         | Description                                                       | Chapter                                      |
| ------------------------------------------------------------------------------ | ----------------------------------------------------------------- | -------------------------------------------- |
| [sui::vec_set](https://docs.sui.io/references/framework/sui/vec_set)           | セット型を実装する                                               | [Collections](./collections)                 |
| [sui::vec_map](https://docs.sui.io/references/framework/sui/vec_map)           | ベクターキーを持つマップを実装する                               | [Collections](./collections)                 |
| [sui::table](https://docs.sui.io/references/framework/sui/table)               | `Table`型とそれと相互作用するメソッドを実装する                   | [Dynamic Collections](./dynamic-collections) |
| [sui::linked_table](https://docs.sui.io/references/framework/sui/linked_table) | `LinkedTable`型とそれと相互作用するメソッドを実装する             | [Dynamic Collections](./dynamic-collections) |
| [sui::bag](https://docs.sui.io/references/framework/sui/bag)                   | `Bag`型とそれと相互作用するメソッドを実装する                     | [Dynamic Collections](./dynamic-collections) |
| [sui::object_table](https://docs.sui.io/references/framework/sui/object_table) | `ObjectTable`型とそれと相互作用するメソッドを実装する             | [Dynamic Collections](./dynamic-collections) |
| [sui::object_bag](https://docs.sui.io/references/framework/sui/object_bag)     | `ObjectBag`型とそれと相互作用するメソッドを実装する               | [Dynamic Collections](./dynamic-collections) |

</div>

## ユーティリティ

<div class="modules-table">

| Module                                                             | Description                                                | Chapter                                 |
| ------------------------------------------------------------------ | ---------------------------------------------------------- | --------------------------------------- |
| [sui::bcs](https://docs.sui.io/references/framework/sui/bcs)       | Implements the BCS encoding and decoding functions         | [Binary Canonical Serialization](./bcs) |
| [sui::borrow](https://docs.sui.io/references/framework/sui/borrow) | Implements the borrowing mechanic for borrowing by _value_ | [Hot Potato](./hot-potato-pattern)      |
| [sui::hex](https://docs.sui.io/references/framework/sui/hex)       | Implements the hex encoding and decoding functions         | -                                       |
| [sui::types](https://docs.sui.io/references/framework/sui/types)   | Provides a way to check if the type is a One-Time-Witness  | [One Time Witness](./one-time-witness)  |

</div>

## エクスポートされたアドレス

Sui Frameworkは、std依存関係から`sui = 0x2`と`std = 0x1`の2つの名前付きアドレスをエクスポートします。

```toml
[addresses]
sui = "0x2"

# MoveStdlib依存関係からエクスポート
std = "0x1"
```

## 暗黙的インポート

[Standard Library](./../move-basics/standard-library#implicit-imports)と同様に、
Sui Frameworkでは一部のモジュールと型が暗黙的にインポートされます。
明示的な`use`インポートなしで利用可能なモジュールと型のリストは以下の通りです：

- sui::object
- sui::object::ID
- sui::object::UID
- sui::tx_context
- sui::tx_context::TxContext
- sui::transfer

## ソースコード

Sui Frameworkのソースコードは
[Sui repository](https://github.com/MystenLabs/sui/tree/main/crates/sui-framework/packages/sui-framework/sources)で
利用可能です。

<!--

Modules:

Coins:
- sui::pay
- sui::sui
- sui::coin
- sui::token
- sui::balance
- sui::deny_list

Commerce:
- sui::kiosk
- sui::display
- sui::kiosk_extension
- sui::transfer_policy


Utilities:
+ sui::bcs
+ sui::hex
- sui::math (deprecated)
+ sui::types
+ sui::borrow


- sui::authenticator

- sui::priority_queue
- sui::table_vec

- sui::url
- sui::versioned

- sui::prover
- sui::random

- sui::bls12381
- sui::ecdsa_k1
- sui::ecdsa_r1
- sui::ecvrf
- sui::ed25519
(also mention verifier 16 growth)
- sui::group_ops
- sui::hash
- sui::hmac
- sui::poseidon
- sui::zklogin_verified_id
- sui::zklogin_verified_issuer

 -->
