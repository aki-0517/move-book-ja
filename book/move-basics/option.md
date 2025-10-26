# Option型

`Option`は、存在するかしないかのオプションの値を表す型です。Moveの`Option`の概念はRustから借用されており、Moveで非常に有用なプリミティブです。`Option`は[標準ライブラリ](./standard-library)で定義されており、以下のように定義されています：

```move
module std::option;

/// Abstraction of a value that may or may not be present.
public struct Option<Element> has copy, drop, store {
    vec: vector<Element>
}
```

_std::option モジュールの[完全なドキュメントはこちら][option-stdlib]を参照してください。_

> `std::option` モジュールはすべてのモジュールに暗黙的にインポートされます。明示的な `use` は不要です。

`Option` 型はジェネリック型で、型パラメータ `Element` を持ちます。内部には `vec` という
単一のフィールドがあり、これは `Element` の `vector` です。このベクターの長さが 0 または 1 となることで、
それぞれ値が存在しない／存在することを表します。

> 注意: `Option` が [enum][enum-reference] ではなく、`vector` を含む `struct` として定義されているのは
> 歴史的な理由によるものです。`Option` は Move に enum が導入される以前に追加されました。

`Option` 型には `Some` と `None` の 2 つのバリアントがあります。`Some` は値を保持し、`None` は値の
不在を表します。`Option` は、空文字や `undefined` のような曖昧な表現を避け、型安全に「値がない」状態を
表すために用いられます。

## 実際の使い方

`Option` 型が必要な理由を示すために、例を考えてみましょう。ユーザー入力を受け取り変数に保存する
アプリケーションでは、必須のフィールドもあれば任意のフィールドもあります。たとえばミドルネームは
任意です。ミドルネームがないことを空文字列で表すこともできますが、その場合、空文字列と未入力を
区別するための追加のチェックが必要になります。代わりに、ミドルネームを `Option` 型で表現できます。

```move file=packages/samples/sources/move-basics/option.move anchor=registry

```

上の例では、`middle_name` フィールドは `Option<String>` 型です。これは、`middle_name` フィールドが
`Some` に包まれた `String` 値を持つか、`None` によって明示的に空であるかのいずれかであることを意味します。
`Option` を使うことで、そのフィールドが任意であることが明確になり、空文字列と未入力を区別するための
曖昧さや余分なチェックを避けられます。

## Option値の作成と利用

`Option` 型（および `std::option` モジュール）は Move では暗黙的にインポートされます。つまり、
`use` 文なしで直接 `Option` 型を使えます。

`Option` 型の値は、`option::some` または `option::none` を使って作成できます。`Option` の値は
いくつかの操作もサポートしています（借用については[参照](references#references-1)の章で扱います）。

```move file=packages/samples/sources/move-basics/option.move anchor=usage

```

## 参考資料

- 標準ライブラリの [std::option][option-stdlib]

[enum-reference]: ./../../reference/enums
[option-stdlib]: https://docs.sui.io/references/framework/std/option
