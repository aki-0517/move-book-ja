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

## 演算子

算術・論理・ビット演算子は、値に対して操作を行うために使用されます。これらの操作は値を
生成するため、式と見なされます。

```move file=packages/samples/sources/move-basics/expression.move anchor=operators

```

## ブロック

ブロックは、中括弧 `{}` で囲まれた文と式の並びです。ブロックは、ブロック内の最後の式の値を
返します（この最後の式には末尾のセミコロンが付いていてはいけません）。ブロック自体も式
なので、式が期待されるあらゆる場所で使用できます。

```move file=packages/samples/sources/move-basics/expression.move anchor=block

```

## 関数呼び出し

関数についての詳細は [関数](./function) セクションで説明します。ただし、これまでのセクションでも
既に関数呼び出しを使用しているため、ここで簡単に触れておきます。関数呼び出しは関数を呼び出し、
その関数本体の最後の式の値を返す式です（最後の式にセミコロンが付いていない場合）。

```move file=packages/samples/sources/move-basics/expression.move anchor=fun_call

```

## 制御フロー式

制御フロー式はプログラムの流れを制御するために使用されます。これらも式であるため、値を返します。
[制御フロー](./control-flow) セクションで詳しく扱います。以下はごく簡単な概要です。

```move file=packages/samples/sources/move-basics/expression.move anchor=control_flow

```
