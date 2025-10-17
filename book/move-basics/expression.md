# 式

プログラミング言語において、式は値を返すコードの単位です。Moveでは、宣言である`let`文を唯一の例外として、ほぼすべてが式です。このセクションでは、式の種類を取り上げ、スコープの概念を紹介します。

> 式はセミコロン`;`で連続されます。セミコロンの後に「式がない」場合、コンパイラは空の式を表す`unit ()`を挿入します。

## リテラル

[プリミティブ型](./primitive-types)セクションで、Moveの基本型を紹介しました。それらを説明するために、リテラルを使用しました。リテラルは、ソースコードで固定値を表現するための記法です。リテラルは変数を初期化したり、固定値を関数の引数として直接渡したりするために使用できます。Moveには以下のリテラルがあります：

- ブール値：`true`と`false`
- 整数値：`0`、`1`、`123123`
- 16進数値：整数を表すために0xで始まる数値、例えば`0x0`、`0x1`、`0x123`
- バイトベクター値：`b`で始まる、例えば`b"bytes_vector"`
- バイト値：`x`で始まる16進数リテラル、例えば`x"0A"`

```move file=packages/samples/sources/move-basics/expression.move anchor=literals

```

## Operators

Arithmetic, logical, and bitwise operators are used to perform operations on values. Since these
operations produce values, they are considered expressions.

```move file=packages/samples/sources/move-basics/expression.move anchor=operators

```

## Blocks

A block is a sequence of statements and expressions enclosed in curly braces `{}`. It returns the
value of the last expression in the block (note that this final expression must not have an ending
semicolon). A block is an expression, so it can be used anywhere an expression is expected.

```move file=packages/samples/sources/move-basics/expression.move anchor=block

```

## Function Calls

We go into detail about functions in the [Functions](./function) section. However, we have already
used function calls in previous sections, so it's worth mentioning them here. A function call is an
expression that calls a function and returns the value of the last expression in the function body,
provided the last expression does not have a terminating semi-colon.

```move file=packages/samples/sources/move-basics/expression.move anchor=fun_call

```

## Control Flow Expressions

Control flow expressions are used to control the flow of the program. They are also expressions, so
they return a value. We cover control flow expressions in the [Control Flow](./control-flow)
section. Here's a very brief overview:

```move file=packages/samples/sources/move-basics/expression.move anchor=control_flow

```
