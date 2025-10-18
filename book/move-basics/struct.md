# Structでのカスタム型

Moveの型システムはカスタム型の定義において光ります。ユーザー定義型は、データレベルだけでなく、その動作においてもアプリケーションの特定のニーズにカスタマイズできます。このセクションでは、構造体の定義とその使用方法を紹介します。

## 構造体

カスタム型を定義するには、`struct`キーワードの後に型の名前を続けます。名前の後に、構造体のフィールドを定義できます。各フィールドは`field_name: field_type`構文で定義されます。フィールド定義はカンマで区切る必要があります。フィールドは他の構造体を含む任意の型であることができます。

> Moveは再帰的な構造体をサポートしません。つまり、構造体は自分自身をフィールドとして含むことはできません。

```move file=packages/samples/sources/move-basics/struct.move anchor=def

```

上の例では、5つのフィールドを持つ `Record` 構造体を定義しています。`title` フィールドは
`String` 型、`artist` フィールドは `Artist` 型、`year` フィールドは `u16` 型、`is_debut`
フィールドは `bool` 型、`edition` フィールドは `Option<u16>` 型です。`edition` フィールドを
`Option<u16>` 型にしているのは、版情報が省略可能であることを表すためです。

構造体はデフォルトでプライベートであり、定義されたモジュールの外部からインポートして
使用することはできません。フィールドもプライベートで、モジュール外からはアクセスできません。
さまざまな可視性修飾子の詳細は [visibility](./visibility) を参照してください。

> 構造体のフィールドはプライベートであり、アクセスできるのはその構造体を定義したモジュール
> のみです。他のモジュールからフィールドを読み書きするには、構造体を定義したモジュールが
> そのフィールドにアクセスするための公開関数を提供している必要があります。

## インスタンスの作成と利用

ここまでは構造体の「定義」を説明しました。次に、構造体の初期化方法とその使い方を見ていきます。
構造体は `struct_name { field1: value1, field2: value2, ... }` 構文で初期化できます。
フィールドの順序は任意ですが、必須のフィールドはすべて設定する必要があります。

```move file=packages/samples/sources/move-basics/struct.move anchor=pack

```

上の例では、`Artist` 構造体のインスタンスを作成し、`name` フィールドに文字列 "The Beatles"
を設定しています。

構造体のフィールドにアクセスするには、`.` 演算子に続けてフィールド名を指定します。

```move file=packages/samples/sources/move-basics/struct.move anchor=access

```

フィールドへのアクセス（可変・不変の双方）は、その構造体を定義したモジュールに限られます。
したがって、上記のコードは `Artist` 構造体と同じモジュール内に置く必要があります。

<!-- ## Accessing Fields

Struct fields are private and can be accessed only by the module defining the struct. To access the fields of a struct, you can use the `.` operator followed by the field name.

```move
# anchor: access file=packages/samples/sources/move-basics/struct.move anchor=access
```
-->

## 構造体のアンパック

構造体はデフォルトで破棄不可です。つまり、初期化した構造体の値は保存するか、アンパックして
使用しなければなりません。構造体のアンパックとは、その値をフィールドに分解することを指します。
これは `let` キーワードの後に構造体名とフィールド名を並べることで行います。

```move file=packages/samples/sources/move-basics/struct.move anchor=unpack

```

上の例では `Artist` 構造体をアンパックし、`name` フィールドの値で新しい変数 `name` を作成しています。
この変数は使用されていないため、コンパイラは警告を出します。警告を抑制するには、アンダースコア `_`
を使って、その変数が意図的に未使用であることを示せます。

```move file=packages/samples/sources/move-basics/struct.move anchor=unpack_ignore

```

## 参考資料

- Moveリファレンスの [Structs](./../../reference/structs)
