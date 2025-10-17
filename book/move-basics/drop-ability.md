# アビリティ：Drop

<!-- TODO: reiterate, given that we introduce abilities one by one -->

<!-- TODO:

- introduce abilities first
- mention them all
- then do one by one

consistency: we / I / you ?
who is we? I am alone, there's no one else here


-->

<!--

// Shall we only talk about `drop` ?
// So that we don't explain scopes and `copy` / `move` semantics just yet?

Chapter: Basic Syntax
Goal: Introduce Copy and Drop abilities of Move. Follows the `struct` section
Notes:
    - compare them to primitive types introduces before;
    - what is an ability without drop
    - drop is not necessary for unpacking
    - make a joke about a bacteria pattern in the code
    - mention that a struct with only `drop` ability is called a Witness
    - mention that a struct without abilities is called a Hot Potato
    - mention that there are two more abilities which are covered in a later chapter

Links:
    - language reference (abilities)
    - authorization patterns (or witness)
    - hot potato pattern
    - key and store abilities (later chapter)

 -->

`drop`アビリティは最もシンプルなアビリティで、構造体のインスタンスを_無視_または_破棄_することを許可します。多くのプログラミング言語では、この動作はデフォルトと見なされます。しかし、Moveでは、`drop`アビリティを持たない構造体を無視することは許可されません。これはMove言語の安全機能であり、すべての資産が適切に処理されることを保証します。`drop`アビリティを持たない構造体を無視しようとすると、コンパイルエラーが発生します。

```move file=packages/samples/sources/move-basics/drop-ability.move anchor=main

```

`drop`アビリティは、カスタムコレクション型で、コレクションが不要になった際の特別な処理を不要にするためによく使用されます。たとえば、`vector`型は`drop`アビリティを持ち、ベクターが不要になった際に無視することを許可します。しかし、Moveの型システムの最大の特徴は、`drop`を持たない能力です。これにより、資産が適切に処理され、無視されないことが保証されます。

`drop`アビリティのみを持つ構造体は_Witness_と呼ばれます。_Witness_の概念は[Witness and Abstract Implementation](./../programmability/witness-pattern)セクションで説明します。

## `drop`アビリティを持つ型

Moveのすべてのネイティブ型は`drop`アビリティを持ちます。これには以下が含まれます：

- [`bool`](./../move-basics/primitive-types#booleans)
- [符号なし整数](./../move-basics/primitive-types#integer-types)
- [`vector<T>`](./../move-basics/vector) で`T`が`drop`を持つ場合
- [`address`](./../move-basics/address)

標準ライブラリで定義されているすべての型も`drop`アビリティを持ちます。これには以下が含まれます：

- [`Option<T>`](./../move-basics/option) で`T`が`drop`を持つ場合
- [`String`](./../move-basics/string)
- [`TypeName`](./../move-basics/type-reflection)

## さらなる読み物

- Move Referenceの[Type Abilities](./../../reference/abilities)
