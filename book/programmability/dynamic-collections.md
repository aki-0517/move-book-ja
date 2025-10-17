# 動的コレクション

[Sui Framework](./sui-framework)は、[動的フィールド](./dynamic-fields)と
[動的オブジェクトフィールド](./dynamic-object-fields)の概念に基づいて構築された
様々なコレクション型を提供しています。これらのコレクションは、動的フィールドとオブジェクトを
格納および管理するためのより安全で理解しやすい方法として設計されています。

各コレクション型について、使用するプリミティブと提供する特定の機能を指定します。

> UIDで動作する動的（オブジェクト）フィールドとは異なり、コレクション型には独自の型があり、
> [関連関数](./../move-basics/struct-methods)の呼び出しが可能です。

## 共通概念

すべてのコレクション型は同じメソッドセットを共有しており、それらは以下の通りです：

- `add` - コレクションにフィールドを追加する
- `remove` - コレクションからフィールドを削除する
- `borrow` - コレクションからフィールドを借用する
- `borrow_mut` - コレクションからフィールドへの可変参照を借用する
- `contains` - フィールドがコレクションに存在するかチェックする
- `length` - コレクション内のフィールド数を返す
- `is_empty` - `length`が0かどうかチェックする

すべてのコレクション型は`borrow`と`borrow_mut`メソッドのインデックス構文をサポートします。
例で角括弧を見る場合、それらは`borrow`と`borrow_mut`呼び出しに変換されます。

```move
let hat: &Hat = &bag[b"key"];
let hat_mut: &mut Hat = &mut bag[b"key"];

// 以下と同等
let hat: &Hat = bag.borrow(b"key");
let hat_mut: &mut Hat = bag.borrow_mut(b"key");
```

例では、これらの関数に焦点を当てるのではなく、コレクション型間の違いに焦点を当てます。

## Bag

Bagは、名前が示すように、異種の値の「バッグ」として機能します。これは、
任意のデータを格納できるシンプルな非ジェネリック型です。Bagはフィールド数を追跡し、
空でない場合は破棄できないため、孤立したフィールドを許可することはありません。

```move
module sui::bag;

public struct Bag has key, store {
    /// このバッグのID
    id: UID,
    /// バッグ内のキーと値のペアの数
    size: u64,
}
```

_[sui::bag][bag-framework]モジュールの完全なドキュメントを参照してください。_

Bagが任意の型を格納するため、提供する追加メソッドは以下の通りです：

- `contains_with_type` - 特定の型でフィールドが存在するかチェックする

構造体フィールドとして使用：

```move file=packages/samples/sources/programmability/dynamic-collections.move anchor=bag_struct

```

Bagの使用：

```move file=packages/samples/sources/programmability/dynamic-collections.move anchor=bag_usage

```

## ObjectBag

`sui::object_bag`モジュールで定義されています。[Bag](#bag)と同一ですが、
内部的に[動的オブジェクトフィールド](./dynamic-object-fields)を使用します。
値としてオブジェクトのみを格納できます。

_[sui::object_bag][object-bag-framework]モジュールの完全なドキュメントを参照してください。_

## Table

Tableは、キーと値に固定型を持つ型付き動的コレクションです。
`sui::table`モジュールで定義されています。

```move
module sui::table;

public struct Table<phantom K: copy + drop + store, phantom V: store> has key, store {
    /// このテーブルのID
    id: UID,
    /// テーブル内のキーと値のペアの数
    size: u64,
}
```

_[sui::table][table-framework]モジュールの完全なドキュメントを参照してください。_

構造体フィールドとして使用：

```move file=packages/samples/sources/programmability/dynamic-collections.move anchor=table_struct

```

Tableの使用：

```move file=packages/samples/sources/programmability/dynamic-collections.move anchor=table_usage

```

## ObjectTable

`sui::object_table`モジュールで定義されています。[Table](#table)と同一ですが、
内部的に[動的オブジェクトフィールド](./dynamic-object-fields)を使用します。
値としてオブジェクトのみを格納できます。

_[sui::object_table][object-table-framework]モジュールの完全なドキュメントを参照してください。_

## LinkedTable

`sui::linked_table`モジュールで定義されており、[Table](#table)と似ていますが、
値がリンクされており、順序付きの挿入と削除が可能です。

```move
module sui::linked_table;

public struct LinkedTable<K: copy + drop + store, phantom V: store> has key, store {
    /// このテーブルのID
    id: UID,
    /// テーブル内のキーと値のペアの数
    size: u64,
    /// テーブルの先頭、つまり最初のエントリのキー
    head: Option<K>,
    /// テーブルの末尾、つまり最後のエントリのキー
    tail: Option<K>,
}
```

_[sui::linked_table][linked-table-framework]モジュールの完全なドキュメントを参照してください。_

LinkedTableに格納された値がリンクされているため、追加と削除のための固有のメソッドがあります。

- `push_front` - テーブルの先頭にキーと値のペアを挿入する
- `push_back` - テーブルの末尾にキーと値のペアを挿入する
- `remove` - キーでキーと値のペアを削除し、値を返す
- `pop_front` - テーブルの先頭を削除し、キーと値を返す
- `pop_back` - テーブルの末尾を削除し、キーと値を返す

構造体フィールドとして使用：

```move file=packages/samples/sources/programmability/dynamic-collections.move anchor=linked_table_struct

```

LinkedTableの使用：

```move file=packages/samples/sources/programmability/dynamic-collections.move anchor=linked_table_usage

```

## まとめ

- [Bag](#bag) - 任意の型のデータを格納できるシンプルなコレクション。
- [ObjectBag](#objectbag) - オブジェクトのみを格納できるコレクション。
- [Table](#table) - キーと値に固定型を持つ型付き動的コレクション。
- [ObjectTable](#objecttable) - Tableと同じですが、オブジェクトのみを格納できます。
- [LinkedTable](#linkedtable) - Tableと似ていますが、値がリンクされています。

## さらなる読み物

- [sui::table][table-framework]モジュールドキュメント。
- [sui::object_table][object-table-framework]モジュールドキュメント。
- [sui::linked_table][linked-table-framework]モジュールドキュメント。
- [sui::bag][bag-framework]モジュールドキュメント。
- [sui::object_bag][object-bag-framework]モジュールドキュメント。

[table-framework]: https://docs.sui.io/references/framework/sui/table
[object-table-framework]: https://docs.sui.io/references/framework/sui/object_table
[linked-table-framework]: https://docs.sui.io/references/framework/sui/linked_table
[bag-framework]: https://docs.sui.io/references/framework/sui/bag
[object-bag-framework]: https://docs.sui.io/references/framework/sui/object_bag

<!-- TODO! -->

<!-- ## Choosing a Collection Type

Depending on the needs of your project, you may choose to -->
