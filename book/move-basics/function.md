# 関数

関数はMoveプログラムの構成要素です。[ユーザートランザクション](./../concepts/what-is-a-transaction)や他の関数から呼び出され、実行可能なコードを再利用可能な単位にグループ化します。関数は引数を取り、値を返すことができます。関数はモジュールレベルで`fun`キーワードで宣言されます。他のモジュールメンバーと同様に、デフォルトではプライベートであり、モジュール内からのみアクセス可能です。

```move file=packages/samples/sources/move-basics/function.move anchor=math

```

この例では、`u64`型の2つの引数を取り、それらの和を返す`add`関数を定義しています。同じモジュール内にある`test_add`関数は、`add`を呼び出すテスト関数です。テストは`assert!`マクロを使用して`add`の結果と期待値を比較します。`assert!`内の条件がfalseに評価された場合、実行は自動的に中止されます。

## 関数宣言

> Moveでは、関数は通常`snake_case`規則を使用して命名されます。これは、関数名がすべて小文字で、単語がアンダースコアで区切られることを意味します。例には`do_something`、`add`、`get_balance`、`is_authorized`などがあります。

関数は`fun`キーワードの後に、関数名（有効なMove識別子）、括弧内の引数リスト、戻り値の型を続けて宣言されます。関数本体は、文と式のシーケンスを含むコードブロックです。関数本体の最後の式が関数の戻り値です。

```move file=packages/samples/sources/move-basics/function.move anchor=return_nothing

```

## 関数へのアクセス

他のモジュールメンバーと同様に、関数はパスを使用してインポートし、アクセスできます。パスはモジュールパスと関数名で構成され、`::`で区切られます。例えば、`book`パッケージ内の`math`モジュールに`add`という名前の関数がある場合、その完全なパスは`book::math::add`になります。モジュールがすでにインポートされている場合、以下の例のように`math::add`として直接アクセスできます：

```move file=packages/samples/sources/move-basics/function_use.move anchor=use_math

```

## 複数の戻り値

Move関数は複数の値を返すことができ、関数から複数のデータを返す必要がある場合に特に有用です。戻り値の型は型のタプルとして指定され、戻り値は式のタプルとして提供されます：

```move file=packages/samples/sources/move-basics/function.move anchor=tuple_return

```

タプルを返す関数呼び出しの結果は、`let (tuple)`構文を使用して変数にアンパックする必要があります：

```move file=packages/samples/sources/move-basics/function.move anchor=tuple_return_imm

```

宣言された値のいずれかが可変として宣言される必要がある場合、`mut`キーワードは変数名の前に配置されます：

```move file=packages/samples/sources/move-basics/function.move anchor=tuple_return_mut

```

引数の一部が使用されない場合、`_`記号を使用して無視できます：

```move file=packages/samples/sources/move-basics/function.move anchor=tuple_return_ignore

```

## 参考資料

- [Functions](./../../reference/functions) in the Move Reference.
