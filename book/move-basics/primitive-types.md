# プリミティブ型

<!-- TODO: Shall we split this into two pages? Maybe give an overview and focus more on specifics? -->

シンプルな値に対して、Moveには多数の組み込みプリミティブ型があります。これらは他のすべての型の基礎です。プリミティブ型は以下の通りです：

- [ブール型](#booleans)
- [符号なし整数](#integer-types)
- [アドレス](./address) - 次のセクションで説明

プリミティブ型に入る前に、まずMoveで変数を宣言し代入する方法を見てみましょう。

## 変数と代入

変数は`let`キーワードを使用して宣言されます。デフォルトでは不変ですが、`mut`キーワードを追加することで可変にすることができます：

```
let <variable_name>[: <type>]  = <expression>;
let mut <variable_name>[: <type>] = <expression>;
```

ここで：

- `<variable_name>` - 変数の名前
- `<type>` - 変数の型、オプション
- `<expression>` - 変数に代入される値

```move file=packages/samples/sources/move-basics/primitive-types.move anchor=variables_and_assignment

```

可変変数は`=`演算子を使用して再代入できます。

```move
y = 43;
```

変数は再宣言によってシャドウ（隠蔽）することもできます。

```move file=packages/samples/sources/move-basics/primitive-types.move anchor=shadowing

```

## ブール型

`bool`型はブール値を表します - はいまたはいいえ、真または偽です。2つの可能な値があります：
`true`と`false`で、これらはMoveのキーワードです。ブール型の場合、コンパイラは常に値から型を推論できるため、明示的に指定する必要はありません。

```move file=packages/samples/sources/move-basics/primitive-types.move anchor=boolean

```

ブール型は、フラグを保存したり、プログラムの流れを制御したりするためによく使用されます。詳細については[制御フロー](./control-flow)セクションを参照してください。

## 整数型

Moveは8ビットから256ビットまでの様々なサイズの符号なし整数をサポートしています。整数型は以下の通りです：

- `u8` - 8ビット
- `u16` - 16ビット
- `u32` - 32ビット
- `u64` - 64ビット
- `u128` - 128ビット
- `u256` - 256ビット

```move file=packages/samples/sources/move-basics/primitive-types.move anchor=integers

```

`true`や`false`のようなブールリテラルは明らかにブール型ですが、`42`のような整数リテラルは任意の整数型にすることができます。ほとんどの場合、コンパイラは値から型を推論し、通常は`u64`をデフォルトとします。しかし、時々コンパイラが型を推論できず、明示的な型注釈が必要になります。これは代入時に提供するか、型サフィックスを使用して提供できます。

```move file=packages/samples/sources/move-basics/primitive-types.move anchor=integer_explicit_type

```

### 演算

Moveは整数の標準的な算術演算をサポートしています：加算、減算、乗算、除算、剰余（余り）です。これらの演算の構文は以下の通りです：

| 構文 | 演算           | 中止条件                                |
| ------ | ------------------- | ---------------------------------------- |
| +      | 加算            | 結果が整数型に対して大きすぎる |
| -      | 減算         | 結果がゼロ未満                 |
| \*     | 乗算      | 結果が整数型に対して大きすぎる |
| %      | 剰余（余り） | 除数が0                         |
| /      | 切り捨て除算 | 除数が0                         |

> ビット演算を含むより多くの演算については、
> [Move Reference](./../../reference/primitive-types/integers#bitwise)を参照してください。

オペランドの型は_一致する必要があり_、そうでなければコンパイラはエラーを発生させます。演算の結果はオペランドと同じ型になります。異なる型で演算を実行するには、オペランドを同じ型にキャストする必要があります。

<!-- TODO: add examples + parentheses for arithmetic operations -->
<!-- TODO: add bitwise operators -->

### `as`によるキャスト

Moveは整数型間の明示的なキャストをサポートしています。構文は以下の通りです：

```move
<expression> as <type>
```

曖昧さを防ぐために、式の周りに括弧が必要な場合があることに注意してください：

```move file=packages/samples/sources/move-basics/primitive-types.move anchor=cast_as

```

オーバーフローを防ぐより複雑な例：

```move file=packages/samples/sources/move-basics/primitive-types.move anchor=overflow

```

### オーバーフロー

Moveはオーバーフロー/アンダーフローをサポートしていません。型の範囲外の値になる演算は実行時エラーを発生させます。これは予期しない動作を防ぐための安全機能です。

```move
let x = 255u8;
let y = 1u8;

// これはエラーを発生させます
let z = x + y;
```

## 参考資料

- [Bool](./../../reference/primitive-types/bool) in the Move Reference.
- [Integer](./../../reference/primitive-types/integers) in the Move Reference.
