# 標準ライブラリ

<!-- Move標準ライブラリは一連のモジュールを提供します -->

Move標準ライブラリは、ネイティブ型と操作のための機能を提供します。ストレージと相互作用しないが、データの操作と処理のための基本的なツールを提供するモジュールの標準コレクションです。これは[Sui Framework](./../programmability/sui-framework)の唯一の依存関係であり、それと一緒にインポートされます。

## 最も一般的なモジュール

この本では、標準ライブラリのほとんどのモジュールについて詳しく説明しますが、利用可能な機能の概要を提供し、どのモジュールがそれを実装しているかを理解できるようにすることも有用です。

<!-- テーマ/custom.cssでのカスタムCSS追加 -->
<div class="modules-table">

| モジュール                                                                           | 説明                                                                | 章                              |
| -------------------------------------------------------------------------------- | -------------------------------------------------------------------------- | ------------------------------------ |
| [std::string](https://docs.sui.io/references/framework/std/string)               | 基本的な文字列操作を提供                                           | [String](./string)                   |
| [std::ascii](https://docs.sui.io/references/framework/std/ascii)                 | 基本的なASCII操作を提供                                            | -                                    |
| [std::option](https://docs.sui.io/references/framework/std/option)               | `Option<T>`を実装                                                     | [Option](./option)                   |
| [std::vector](https://docs.sui.io/references/framework/std/vector)               | ベクター型のネイティブ操作                                       | [Vector](./vector)                   |
| [std::bcs](https://docs.sui.io/references/framework/std/bcs)                     | `bcs::to_bytes()`関数を含む                                    | [BCS](./../programmability/bcs)      |
| [std::address](https://docs.sui.io/references/framework/std/address)             | 単一の`address::length`関数を含む                               | [Address](./address)                 |
| [std::type_name](https://docs.sui.io/references/framework/std/type_name)         | ランタイム_型リフレクション_を可能にする                                           | [Type Reflection](./type-reflection) |
| [std::hash](https://docs.sui.io/references/framework/std/hash)                   | ハッシュ関数：`sha2_256`と`sha3_256`                               | -                                    |
| [std::debug](https://docs.sui.io/references/framework/std/debug)                 | デバッグ関数を含む（**テスト**モードでのみ利用可能） | -                                    |
| [std::bit_vector](https://docs.sui.io/references/framework/std/bit_vector)       | ビットベクターの操作を提供                                         | -                                    |
| [std::fixed_point32](https://docs.sui.io/references/framework/std/fixed_point32) | `FixedPoint32`型を提供                                           | -                                    |

</div>

## 整数モジュール

Move標準ライブラリは、整数型に関連する一連の関数を提供します。これらの関数は複数のモジュールに分割され、それぞれが特定の整数型に関連付けられています。これらのモジュールは直接インポートすべきではありません。なぜなら、それらの関数はすべての整数値で利用可能だからです。

> すべてのモジュールは同じ関数セットを提供します。すなわち、`max`、`diff`、
> `divide_and_round_up`、`sqrt`、`pow`です。

<!-- テーマ/custom.cssでのカスタムCSS追加 -->
<div class="modules-table">

| モジュール                                                         | 説明                   |
| -------------------------------------------------------------- | ----------------------------- |
| [std::u8](https://docs.sui.io/references/framework/std/u8)     | `u8`型の関数   |
| [std::u16](https://docs.sui.io/references/framework/std/u16)   | `u16`型の関数  |
| [std::u32](https://docs.sui.io/references/framework/std/u32)   | `u32`型の関数  |
| [std::u64](https://docs.sui.io/references/framework/std/u64)   | `u64`型の関数  |
| [std::u128](https://docs.sui.io/references/framework/std/u128) | `u128`型の関数 |
| [std::u256](https://docs.sui.io/references/framework/std/u256) | `u256`型の関数 |

</div>

## エクスポートされたアドレス

標準ライブラリは単一の名前付きアドレス - `std = 0x1`をエクスポートします。ここでエイリアス`std`が定義されていることに注意してください。

```toml
[addresses]
std = "0x1"
```

## 暗黙的インポート

一部のモジュールは暗黙的にインポートされ、明示的な`use`インポートなしでモジュール内で利用可能です。標準ライブラリの場合、これらのモジュールと型には以下が含まれます：

- std::vector
- std::option
- std::option::Option

## Sui Frameworkなしでstdをインポート

Move標準ライブラリはパッケージに直接インポートできます。しかし、`std`だけでは意味のあるアプリケーションを構築するには不十分です。なぜなら、ストレージ機能を提供せず、オンチェーン状態と相互作用できないからです。

```toml
MoveStdlib = { git = "https://github.com/MystenLabs/sui.git", subdir = "crates/sui-framework/packages/move-stdlib", rev = "framework/mainnet" }
```

## ソースコード

Move標準ライブラリのソースコードは[Suiリポジトリ](https://github.com/MystenLabs/sui/tree/main/crates/sui-framework/packages/move-stdlib/sources)で利用可能です。