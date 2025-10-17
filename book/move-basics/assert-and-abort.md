# 実行の中止

<!-- Consider "aborting execution" -->

<!--

Chapter: Basic Syntax
Goal: Introduce abort keyword and `assert!` macro.
Notes:
    - previous chapter mentions constants
    - error constants standard ECamelCase
    - `assert!` macro
    - asserts should go before the main logic
    - Move has no catch mechanism
    - abort codes are local to the module
    - there are no error messages emitted
    - error codes should handle all possible scenarios in this module

Links:
    - constants (previous section)
 -->

トランザクションは成功するか失敗するかのどちらかです。実行が成功すると、オブジェクトとオンチェーンデータに加えられたすべての変更が適用され、トランザクションがブロックチェーンにコミットされます。一方、トランザクションが中止された場合、変更は適用されません。`abort`キーワードを使用してトランザクションを中止し、行われた変更を元に戻します。

> Moveにはキャッチメカニズムがないことに注意することが重要です。トランザクションが中止されると、これまでに行われた変更は元に戻され、トランザクションは失敗したと見なされます。

## Abort

`abort`キーワードは、トランザクションの実行を中止するために使用されます。これは中止コードと組み合わせて使用され、中止コードはトランザクションの呼び出し元に返されます。中止コードは`u64`型の[整数](./primitive-types)です。

```move file=packages/samples/sources/move-basics/assert-and-abort.move anchor=abort

```

上記のコードは、当然ながら中止コード`1`で中止されます。

## assert!

`assert!`マクロは、条件をアサートするために使用できる組み込みマクロです。条件がfalseの場合、トランザクションは指定された中止コードで中止されます。`assert!`マクロは、条件が満たされない場合にトランザクションを中止する便利な方法です。このマクロは、`if`式＋`abort`で書かれるコードを短縮します。`code`引数はオプションですが、`u64`値または`#[error]`（詳細については以下を参照）である必要があります。

```move file=packages/samples/sources/move-basics/assert-and-abort.move anchor=assert

```

## エラー定数

エラーコードをより説明的にするために、[エラー定数](./constants)を定義することは良い習慣です。エラー定数は`const`宣言として定義され、通常は`E`で始まり、その後にキャメルケースの名前が続きます。エラー定数は他の定数と似ており、特別な処理はありません。しかし、コードの可読性を向上させ、中止シナリオを理解しやすくするために一般的に使用されます。

```move file=packages/samples/sources/move-basics/assert-and-abort.move anchor=error_const

```

## エラーメッセージ

Move 2024では、`#[error]`属性でマークされた特別なタイプのエラー定数が導入されました。この属性により、エラー定数は`vector<u8>`型にすることができ、エラーメッセージを格納するために使用できます。

```move file=packages/samples/sources/move-basics/assert-and-abort.move anchor=error_attribute

```

## さらなる読み物

- Move Referenceの[Abort and Assert](./../../reference/abort-and-assert)
- Moveでのエラーハンドリングのベストプラクティスについて学ぶため、[Better Error Handling](./../guides/better-error-handling)ガイドを読むことをお勧めします。
