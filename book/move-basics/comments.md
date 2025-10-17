# コメント

<!--

Chapter: Basic Syntax
Goal: Introduce comments.
Notes:
    - doc comments are used in docgen
    - only public members are documented
    - doc comments are placed in between attributes and the definition
    - doc comments are allowed for: modules, structs, functions, constants
    - give an example of how doc comments are translated
 -->

コメントは、コードにメモやドキュメントを追加する方法です。コメントはコンパイラによって無視され、Moveバイトコードには変換されません。コメントを使用して、コードが何をするかを説明したり、自分や他の開発者にメモを残したり、コードの一部を一時的に削除したり、ドキュメントを生成したりできます。Moveには3種類のコメントがあります：行コメント、ブロックコメント、ドキュメントコメント。

## 行コメント

ダブルスラッシュ`//`を使用して、行の残りをコメントアウトできます。`//`以降のすべてはコンパイラによって無視されます。

```move file=packages/samples/sources/move-basics/comments-line.move anchor=main

```

## ブロックコメント

ブロックコメントは、コードのブロックをコメントアウトするために使用されます。`/*`で始まり`*/`で終わります。`/*`と`*/`の間のすべてはコンパイラによって無視されます。ブロックコメントを使用して、1行または複数行をコメントアウトできます。行の一部をコメントアウトすることもできます。

```move file=packages/samples/sources/move-basics/comments-block.move anchor=main

```

この例は少し極端ですが、ブロックコメントのすべての使用方法を示しています。

## ドキュメントコメント

ドキュメントコメントは、コードのドキュメントを生成するために使用される特別なコメントです。ブロックコメントに似ていますが、3つのスラッシュ`///`で始まり、ドキュメント化するアイテムの定義の前に配置されます。

```move file=packages/samples/sources/move-basics/comments-doc.move anchor=main

```

## 空白文字

一部の言語とは異なり、空白文字（スペース、タブ、改行）はプログラムの意味に影響しません。

<!-- TODO: docgen, which members are in the documentation -->
