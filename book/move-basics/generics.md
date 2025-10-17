# ジェネリクス

ジェネリクスは、あらゆる型で動作する型や関数を定義する方法です。これは、異なる型で使用できる関数を書きたい場合や、他の任意の型を保持できる型を定義したい場合に有用です。ジェネリクスは、コレクション、抽象実装など、Moveの多くの高度な機能の基盤となっています。

## 標準ライブラリでの使用

この章では既に[vector](./vector)型について説明しましたが、これは他の任意の型を保持できるジェネリック型です。標準ライブラリでのジェネリック型のもう一つの例は[Option](./option)型で、これは存在する場合としない場合がある値を表現するために使用されます。

## ジェネリクス構文

ジェネリック型または関数を定義するには、型シグネチャに角括弧（`<`と`>`）で囲まれたジェネリックパラメータのリストが必要です。ジェネリックパラメータはカンマで区切られます。

```move file=packages/samples/sources/move-basics/generics.move anchor=container

```

上記の例では、`Container`は単一の型パラメータ`T`を持つジェネリック型で、コンテナの`value`フィールドは`T`を格納します。`new`関数は単一の型パラメータ`T`を持つジェネリック関数で、与えられた値を持つ`Container`を返します。ジェネリック型は具象型で初期化する必要があり、ジェネリック関数は具象型で呼び出す必要がありますが、場合によってはMoveコンパイラが正しい型を推論できます。

```move file=packages/samples/sources/move-basics/generics.move anchor=test_container

```

テスト関数`test_container`では、`u8`値を持つ新しい`Container`を作成する3つの同等な方法を示しています。数値定数は曖昧な型を持つため、どこかで数値リテラルの型を指定する必要があります（コンテナの型、`new`のパラメータ、または数値リテラル自体で）。これらの一つを指定すると、コンパイラは他を推論できます。

## 複数の型パラメータ

複数の型パラメータを持つ型や関数を定義できます。型パラメータはカンマで区切られます。

```move file=packages/samples/sources/move-basics/generics.move anchor=pair

```

上記の例では、`Pair`は2つの型パラメータ`T`と`U`を持つジェネリック型で、`new_pair`関数は2つの型パラメータ`T`と`U`を持つジェネリック関数です。この関数は与えられた値を持つ`Pair`を返します。型パラメータの順序は重要で、型シグネチャの型パラメータの順序と一致する必要があります。

```move file=packages/samples/sources/move-basics/generics.move anchor=test_pair

```

`new_pair`関数で型パラメータを入れ替えた別のインスタンスを追加し、2つの型を比較しようとすると、型シグネチャが異なり、比較できないことがわかります。

```move file=packages/samples/sources/move-basics/generics.move anchor=test_pair_swap

```

`pair1`と`pair2`の型が異なるため、比較`pair1 == pair2`はコンパイルされません。

## なぜジェネリクスが必要なのか？

上記の例では、ジェネリック型をインスタンス化し、ジェネリック関数を呼び出してこれらの型のインスタンスを作成することに焦点を当てました。しかし、ジェネリクスの真の力は、基本的なジェネリック型に対して共有の動作を定義し、具象型とは独立してそれを使用できることにあります。これは、コレクション、抽象実装、およびMoveの他の高度な機能を扱う際に特に有用です。

```move file=packages/samples/sources/move-basics/generics.move anchor=user

```

上記の例では、`User`は単一の型パラメータ`T`を持つジェネリック型で、共有フィールド`name`、`age`、および任意の型を格納できるジェネリック`metadata`フィールドを持っています。`metadata`が何であっても、`User`のすべてのインスタンスは同じフィールドとメソッドを含みます。

```move file=packages/samples/sources/move-basics/generics.move anchor=update_user

```

## ファントム型パラメータ

場合によっては、型のフィールドやメソッドで使用されない型パラメータを持つジェネリック型を定義したいことがあります。これは_ファントム型パラメータ_と呼ばれます。ファントム型パラメータは、他の任意の型を保持できる型を定義したいが、型パラメータにいくつかの制約を強制したい場合に有用です。

```move file=packages/samples/sources/move-basics/generics.move anchor=phantom

```

ここでの`Coin`型は、型パラメータ`T`を使用するフィールドやメソッドを含んでいません。これは異なる種類のコインを区別し、型パラメータ`T`にいくつかの制約を強制するために使用されます。

```move file=packages/samples/sources/move-basics/generics.move anchor=test_phantom

```

上記の例では、異なるファントム型パラメータ`USD`と`EUR`を持つ2つの異なる`Coin`インスタンスを作成する方法を示しています。型パラメータ`T`は`Coin`型のフィールドやメソッドでは使用されませんが、異なる種類のコインを区別するために使用されます。これにより、`USD`と`EUR`のコインが誤って混同されないことが保証されます。

## 型パラメータの制約

型パラメータは特定のアビリティを持つように制約できます。これは、内部型が_copy_や_drop_などの特定の動作を許可する必要がある場合に有用です。型パラメータを制約する構文は`T: <ability> + <ability>`です。

```move file=packages/samples/sources/move-basics/generics.move anchor=constraints

```

Moveコンパイラは、型パラメータ`T`が指定されたアビリティを持つことを強制します。型パラメータが指定されたアビリティを持たない場合、コードはコンパイルされません。

<!-- TODO: failure case -->

```move file=packages/samples/sources/move-basics/generics.move anchor=test_constraints

```

## 参考文献

- Move Referenceの[Generics](./../../reference/generics)。
