## The `while` Loop

`while`文は、関連するブール式がtrueと評価される限り、コードブロックを繰り返し実行します。`if`で見たように、ブール式はループの各反復の前に評価されます。また、条件文と同様に、`while`ループは式であり、その後に他の式が続く場合はセミコロンが必要です。

`while`ループの構文は以下の通りです：

```move
while (<bool_expression>) { <expressions>; };
```

非常にシンプルな条件を持つ`while`ループの例を示します：

```move file=packages/samples/sources/move-basics/control-flow.move anchor=while_loop

```

## 無限`loop`

ここで、ブール式が常に`true`であるシナリオを想像してみましょう。例えば、`while`条件に文字通り`true`を渡した場合です。これは`loop`文の動作と似ていますが、`while`は条件を評価する点が異なります。

```move file=packages/samples/sources/move-basics/control-flow.move anchor=infinite_while

```

無限`while`ループ、または常に`true`条件を持つ`while`ループは、`loop`と同等です。`loop`を作成する構文は簡単です：

```move
loop { <expressions>; };
```

`while`の代わりに`loop`を使用して前の例を書き直してみましょう：

```move file=packages/samples/sources/move-basics/control-flow.move anchor=infinite_loop

```

無限ループはMoveではほとんど実用的ではありません。すべての操作はガスを消費し、無限ループは必然的にガスの枯渇につながるからです。ループを使用していることに気づいたら、他の制御フロー構造でより効率的に処理できる場合が多いので、より良いアプローチがないか検討してください。とはいえ、`loop`は`break`と`continue`文と組み合わせて、制御された柔軟なループ動作を作成する際に有用かもしれません。

## ループの早期終了

すでに述べたように、無限ループはそれ自体ではむしろ無用です。そこで`break`と`continue`文を導入します。これらはそれぞれループを早期に終了し、現在の反復の残りをスキップするために使用されます。

`break`文の構文は（セミコロンなし）：

```move
break
```

`break`文はループの実行を停止し、早期に終了するために使用されます。特定の条件が満たされたときにループを終了するために、条件文と組み合わせて使用されることがよくあります。この点を説明するために、前の例の無限`loop`を`while`ループのように見え、動作するものに変えてみましょう：

```move file=packages/samples/sources/move-basics/control-flow.move anchor=break_loop

```

`while`ループとほぼ同じですね？`break`文は`x`が5のときにループを終了するために使用されます。`break`文を削除すると、前の例と同じように、ループは永遠に実行されます。

## 反復のスキップ

`continue`文は、現在の反復の残りをスキップして次の反復を開始するために使用されます。`break`と同様に、特定の条件が満たされたときに反復の残りをスキップするために条件文と組み合わせて使用されます。

`continue`文の構文は（セミコロンなし）：

```move
continue
```

以下の例は奇数をスキップし、0から10の偶数のみを出力します：

```move file=packages/samples/sources/move-basics/control-flow.move anchor=continue_loop

```

`break`と`continue`文は、`while`と`loop`の両方のループで使用できます。

## 早期リターン

`return`文は[関数](./function)を早期に終了し、値を返すために使用されます。特定の条件が満たされたときに関数を終了するために、条件文と組み合わせて使用されることがよくあります。`return`文の構文は：

```move
return <expression>
```

特定の条件が満たされたときに値を返す関数の例を示します：

```move file=packages/samples/sources/move-basics/control-flow.move anchor=return_statement

```

他の多くの言語とは異なり、`return`文は関数内の最後の式には必要ありません。関数ブロック内の最後の式は自動的に返されます。しかし、`return`文は特定の条件が満たされたときに関数を早期に終了したい場合に便利です。

## さらなる読み物

- Move Referenceの[Control Flow](./../../reference/control-flow)章