# Vector

VectorはMoveで要素のコレクションを格納するネイティブな方法です。他のプログラミング言語の配列に似ていますが、いくつかの違いがあります。このセクションでは、`vector`型とその操作を紹介します。

## Vectorの構文

`vector`型は、`vector`キーワードの後に山括弧内に要素の型を記述して書きます。要素の型は、他のvectorを含む任意の有効なMove型であることができます。

Moveにはvectorリテラル構文があり、`vector`キーワードの後に要素を含む角括弧（または空のvectorの場合は要素なし）を使用してvectorを作成できます。

```move file=packages/samples/sources/move-basics/vector.move anchor=literal

```

`vector`型はMoveの組み込み型であり、モジュールからインポートする必要はありません。Vectorの操作は`std::vector`モジュールで定義されており、暗黙的にインポートされ、明示的な`use`インポートなしで直接使用できます。

## Vectorの操作

標準ライブラリはvectorを操作するメソッドを提供します。以下は最もよく使用される操作のいくつかです：

- `push_back`: vectorの末尾に要素を追加します。
- `pop_back`: vectorから最後の要素を削除します。
- `length`: vector内の要素数を返します。
- `is_empty`: vectorが空の場合にtrueを返します。
- `remove`: 指定されたインデックスの要素を削除します。

```move file=packages/samples/sources/move-basics/vector.move anchor=methods

```

## ドロップ不可能な型のVectorの破棄

ドロップ不可能な型のvectorは破棄できません。`drop`アビリティを持たない型のvectorを定義した場合、vectorの値は無視できません。vectorが空の場合、コンパイラは`destroy_empty`関数の明示的な呼び出しを要求します。

```move file=packages/samples/sources/move-basics/vector.move anchor=no_drop

```

`destroy_empty`関数は、空でないvectorに対して呼び出した場合、実行時に失敗します。

## さらなる読み物

- Move Referenceの[Vector](./../../reference/primitive-types/vector)
- [std::vector](https://docs.sui.io/references/framework/std/vector)モジュールドキュメント
