# 参照

<!--

Chapter: Basic Syntax
Goal: Show what the borrow checker is and how it works.
Notes:
    - give the metro pass example
    - show why passing by reference is useful
    - mention that reference comparison is faster
    - references can be both mutable and immutable
    - immutable access to shared objects is faster
    - implicit copy
    - moving the value
    - unpacking a reference (mutable and immutable)

 -->

[所有権とスコープ](./ownership-and-scope)セクションでは、値が関数に渡されるときに、その関数のスコープに_移動_されることを説明しました。これは、関数がその値の所有者になり、元のスコープ（所有者）はもはやそれを使用できないことを意味します。これは、値が同時に複数の場所で使用されないことを保証するMoveの重要な概念です。ただし、関数に値を渡しながら所有権を保持したい場合もあります。ここで参照が登場します。

これを説明するために、簡単な例を考えてみましょう - 地下鉄パスのアプリケーションです。カードが以下の4つの異なるシナリオを見てみます：

1. 固定価格でキオスクで購入
2. 乗客が有効なパスを持っていることを証明するために検査員に提示
3. 地下鉄に入場し乗車料金を支払うために改札機で使用
4. 空になった後にリサイクル

## レイアウト

地下鉄パスアプリケーションの初期レイアウトは簡単です。`Card`型と、1枚のカードでの乗車回数を表す`USES`[定数](./constants)を定義します。また、カードが空の場合とカードが空でない場合の[エラー定数](./assert-and-abort#error-constants)も追加します。

```move file=packages/samples/sources/move-basics/references.move anchor=header_new
module book::metro_pass;


```

<!-- In [the previous section](./ownership-and-scope) we explained the ownership and scope in Move. We showed how the value is *moved* to a new scope, and how it changes the owner. In this section, we will explain how to *borrow* a reference to a value to avoid moving it, and how Move's *borrow checker* ensures that the references are used correctly. -->

## 参照

参照は、所有権を放棄することなく関数に値を_示す_方法です。この場合、カードを検査員に見せるとき、その所有権を放棄したくありませんし、検査員が乗車回数を使い切ることも許可しません。私たちはカードの値の_読み取り_を許可し、その所有権を証明したいだけです。

そのために、関数のシグネチャで`&`シンボルを使用して、値自体ではなく値への_参照_を渡していることを示します。

```move file=packages/samples/sources/move-basics/references.move anchor=immutable

```

関数はカードの所有権を取得しないため、そのデータを_読み取る_ことはできますが、_書き込む_ことはできません。つまり、乗車回数を変更することはできません。さらに、関数のシグネチャは、カードインスタンスなしには呼び出せないことを保証します。これは、次の章で説明する[ケイパビリティパターン](./../programmability/capability)を可能にする重要な性質です。

値への参照を作成することは、しばしば値を「借用」すると呼ばれます。例えば、`Option`でラップされた値への参照を取得するメソッドは`borrow`と呼ばれます。

## 可変参照

場合によっては、関数がカードを変更することを許可したいことがあります。例えば、改札機でカードを使用するときは、乗車回数を減らす必要があります。これを実現するために、関数のシグネチャで`&mut`キーワードを使用します。

```move file=packages/samples/sources/move-basics/references.move anchor=mutable

```

関数本体で見ることができるように、`&mut`参照は値の変更を許可し、関数は乗車回数を消費できます。

## 値渡し

最後に、値自体を関数に渡すときに何が起こるかを説明しましょう。この場合、関数は値の所有権を取得し、元のスコープでアクセスできなくなります。カードの所有者はそれをリサイクルし、それによって所有権を関数に放棄できます。

```move file=packages/samples/sources/move-basics/references.move anchor=move

```

`recycle`関数では、カードは値渡しで渡され、所有権を関数に移譲します。これにより、アンパックして破棄できます。

> 注：Moveでは、`_`は分解で使用されるワイルドカードパターンで、値を消費しながらフィールドを無視するために使用されます。分解は構造体型のすべてのフィールドと一致する必要があります。構造体にフィールドがある場合、すべてを明示的にリストするか、`_`を使用して不要なフィールドを無視する必要があります。

## 完全な例

アプリケーションの完全なフローを説明するために、すべての部分をテストでまとめてみましょう。

```move file=packages/samples/sources/move-basics/references.move anchor=move_2024

```

## 参考文献

- Move Referenceの[References](/reference/primitive-types/references)。

<!-- ## Dereference and Copy -->

<!-- TODO: defer and copy, *& -->

<!-- ## Notes -->

<!--
    Move 2024 is great but it's better to show the example with explicit &t and &mut t
    ...and then say that the example could be rewritten with the new syntax


-->

<!-- ## Move 2024

Here's the test from this page written with the Move 2024 syntax:

```move file=packages/samples/sources/move-basics/references.move anchor=move_2024
```
-->
