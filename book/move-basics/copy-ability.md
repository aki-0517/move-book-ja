# アビリティ：Copy

Moveでは、型の_copy_アビリティは、その型のインスタンスまたは値をコピー、つまり複製できることを示します。この動作は数値や他のプリミティブ型を扱う際にはデフォルトで提供されますが、カスタム型にはデフォルトではありません。Moveはデジタル資産とリソースを表現するように設計されており、リソースを複製する能力を制御することは、リソースモデルの重要な原則です。しかし、Moveの型システムでは、カスタム型に_copy_アビリティを追加することができます：

```move file=packages/samples/sources/move-basics/copy-ability.move anchor=copyable

```

上記の例では、_copy_アビリティを持つカスタム型`Copyable`を定義しています。これは、`Copyable`のインスタンスが暗黙的にも明示的にもコピーできることを意味します。

```move file=packages/samples/sources/move-basics/copy-ability.move anchor=copyable_test

```

上記の例では、`a`は`b`に暗黙的にコピーされ、その後参照外し演算子を使用して`c`に明示的にコピーされます。`Copyable`が_copy_アビリティを持たない場合、コードはコンパイルされず、Moveコンパイラがエラーを発生させます。

> 注：Moveでは、空の括弧での分解は、特にdropアビリティを持たない型において、未使用の変数を消費するためによく使用されます。これにより、値が明示的な使用なしにスコープから外れることによるコンパイラエラーを防ぎます。また、Moveは厳密な型付けと所有権ルールを強制するため、分解において型名（例：`let Copyable {} = a;`の`Copyable`）を要求します。

## コピーとDrop

`copy`アビリティは[`drop`アビリティ](./drop-ability)と密接に関連しています。型が_copy_アビリティを持つ場合、`drop`も持つべき可能性が非常に高いです。これは、インスタンスが不要になった際にリソースをクリーンアップするために_drop_アビリティが必要だからです。型が_copy_のみを持つ場合、インスタンスは明示的に使用または消費される必要があるため、その管理がより複雑になります。

```move file=packages/samples/sources/move-basics/copy-ability.move anchor=copy_drop

```

Moveのすべてのプリミティブ型は、_copy_と_drop_アビリティを持つかのように動作します。これは、それらがコピーおよびドロップでき、Moveコンパイラがメモリ管理を処理することを意味します。

## `copy`アビリティを持つ型

Moveのすべてのネイティブ型は`copy`アビリティを持ちます。これには以下が含まれます：

- [bool](./../move-basics/primitive-types#booleans)
- [符号なし整数](./../move-basics/primitive-types#integer-types)
- [vector](./../move-basics/vector)
- [address](./../move-basics/address)

標準ライブラリで定義されているすべての型も`copy`アビリティを持ちます。これには以下が含まれます：

- [Option](./../move-basics/option)
- [String](./../move-basics/string)
- [TypeName](./../move-basics/type-reflection)

## さらなる読み物

- Move Referenceの[Type Abilities](./../../reference/abilities)
