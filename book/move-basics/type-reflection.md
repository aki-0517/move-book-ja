# 型リフレクション

プログラミング言語において、_リフレクション_はプログラムが自分自身の構造と動作を調査および変更する能力です。Moveは、実行時に値の型を検査できる限定的な形のリフレクションをサポートします。これは、同質コレクションに型情報を格納する必要がある場合や、特定のパッケージから型が来ているかどうかをチェックしたい場合に便利です。

型リフレクションは[標準ライブラリ](./standard-library)モジュール[`std::type_name`][type-name-stdlib]で実装されています。これは関数のセットを提供し、主要なものは`with_defining_ids`と`with_original_ids`です。

```move
let defining_type_name: TypeName = type_name::with_defining_ids<T>();
let original_type_name: TypeName = type_name::with_original_ids<T>();

// Returns only "ID" of the package.
let defining_package: address = type_name::defining_id<T>();
let original_package: address = type_name::original_id<T>();
```

## Defining IDとOriginal ID

_defining ID_と_original ID_の違いを理解することが重要です。

- Original IDはパッケージの最初に公開されたID（最初のアップグレード前）です。
- Defining IDはリフレクションされる型を導入したパッケージIDであり、このプロパティはパッケージのアップグレードで新しい型が導入される際に重要になります。

For example, suppose the first version of a package was published at `0xA` and introduced the type
`Version1`. Later, in an upgrade, the package moved to address `0xB` and introduced a new type
`Version2`. For `Version1`, the defining ID and original ID are the same. For `Version2`, however,
they differ: the original ID is `0xA`, while the defining ID is `0xB`.

```move
// Note: values `0xA` and `0xB` are used for illustration purposes only!
// Don't attempt to run this code, as it will inevitably fail.
module book::upgrade;

// Introduced in initial version.
// Defining ID: 0xA
// Original ID: 0xA
//
// With Defining IDs: 0xA::upgrade::Version1
// With Original IDs: 0xA::upgrade::Version1
public struct Version1 has drop {}

// Introduced in a package upgrade.
// Defining ID: 0xB
// highlight-important
// Original ID: 0xA
//
// With Defining IDs: 0xB::upgrade::Version2
// highlight-important
// With Original IDs: 0xA::upgrade::Version2
public struct Version2 has drop {}
```

## 実際の使用

モジュールは簡単で、結果に対して許可される操作は、文字列表現の取得と型のモジュールおよびアドレスの抽出に限定されます。

```move file=packages/samples/sources/move-basics/type-reflection.move anchor=main

```

## さらなる読み物

- [std::type_name][type-name-stdlib]モジュールドキュメント

[type-name-stdlib]: https://docs.sui.io/references/framework/std/type_name
