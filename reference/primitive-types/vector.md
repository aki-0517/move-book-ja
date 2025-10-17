---
title: 'Vector | リファレンス'
description: ''
---

# Vector

`vector<T>`はMoveによって提供される唯一のプリミティブコレクション型です。`vector<T>`は、末尾から値をプッシュ/ポップすることで成長または縮小できる`T`の同種コレクションです。

`vector<T>`は任意の型`T`でインスタンス化できます。例えば、`vector<u64>`、`vector<address>`、`vector<0x42::my_module::MyData>`、`vector<vector<u8>>`はすべて有効なベクター型です。

## リテラル

### 一般的な`vector`リテラル

任意の型のベクターは`vector`リテラルで作成できます。

| 構文                | 型                                                                          | 説明                                |
| --------------------- | ----------------------------------------------------------------------------- | ------------------------------------------ |
| `vector[]`            | `vector[]: vector<T>`（`T`は任意の単一の非参照型）             | 空のベクター                            |
| `vector[e1, ..., en]` | `vector[e1, ..., en]: vector<T>`（`e_i: T`で`0 < i <= n`かつ`n > 0`） | `n`個の要素を持つベクター（長さ`n`） |

これらの場合、`vector`の型は、要素の型またはベクターの使用法から推論されます。型を推論できない場合、または単に明確性を高めるために、型を明示的に指定できます：

```move
vector<T>[]: vector<T>
vector<T>[e1, ..., en]: vector<T>
```

#### ベクターリテラルの例

```move
(vector[]: vector<bool>);
(vector[0u8, 1u8, 2u8]: vector<u8>);
(vector<u128>[]: vector<u128>);
(vector<address>[@0x42, @0x100]: vector<address>);
```

### `vector<u8>`リテラル

Moveでのベクターの一般的な使用例は、「バイト配列」を表現することです。これは`vector<u8>`で表現されます。これらの値は、公開鍵やハッシュ結果など、暗号化目的でよく使用されます。これらの値は非常に一般的なので、各`u8`値を数値形式で指定する`vector[]`を使用する代わりに、値をより読みやすくするための特定の構文が提供されています。

現在、`vector<u8>`リテラルの2つのサポートされた型があります：_バイト文字列_と_16進文字列_です。

#### バイト文字列

バイト文字列は`b`をプレフィックスとする引用符付き文字列リテラルです。例：`b"Hello!\n"`

これらはエスケープシーケンスを許可するASCIIエンコード文字列です。現在、サポートされているエスケープシーケンスは以下の通りです：

| エスケープシーケンス | 説明                                    |
| --------------- | ---------------------------------------------- |
| `\n`            | 改行（またはラインフィード）                        |
| `\r`            | キャリッジリターン                                |
| `\t`            | タブ                                            |
| `\\`            | バックスラッシュ                                      |
| `\0`            | ヌル                                           |
| `\"`            | 引用符                                          |
| `\xHH`          | 16進エスケープ、16進バイトシーケンス`HH`を挿入 |

#### 16進文字列

16進文字列は`x`をプレフィックスとする引用符付き文字列リテラルです。例：`x"48656C6C6F210A"`

`00`から`FF`までの各バイトペアは、16進エンコードされた`u8`値として解釈されます。したがって、各バイトペアは結果の`vector<u8>`の単一エントリに対応します。

#### 文字列リテラルの例

```move
fun byte_and_hex_strings() {
    assert!(b"" == x"", 0);
    assert!(b"Hello!\n" == x"48656C6C6F210A", 1);
    assert!(b"\x48\x65\x6C\x6C\x6F\x21\x0A" == x"48656C6C6F210A", 2);
    assert!(
        b"\"Hello\tworld!\"\n \r \\Null=\0" ==
            x"2248656C6C6F09776F726C6421220A200D205C4E756C6C3D00",
        3
    );
}
```

## 操作

`vector`は、Move標準ライブラリの`std::vector`モジュールを通じて以下の操作をサポートします：

| 関数                                                   | 説明                                                                                                                                                     | アボート？                        |
| ---------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------ |
| `vector::empty<T>(): vector<T>`                            | 型`T`の値を格納できる空のベクターを作成                                                                                                        | なし                          |
| `vector::singleton<T>(t: T): vector<T>`                    | `t`を含むサイズ1のベクターを作成                                                                                                                        | なし                          |
| `vector::push_back<T>(v: &mut vector<T>, t: T)`            | `t`を`v`の末尾に追加                                                                                                                                       | なし                          |
| `vector::pop_back<T>(v: &mut vector<T>): T`                | `v`の最後の要素を削除して返す                                                                                                                       | `v`が空の場合                |
| `vector::borrow<T>(v: &vector<T>, i: u64): &T`             | インデックス`i`の`T`への不変参照を返す                                                                                                           | `i`が範囲外の場合        |
| `vector::borrow_mut<T>(v: &mut vector<T>, i: u64): &mut T` | インデックス`i`の`T`への可変参照を返す                                                                                                              | `i`が範囲外の場合        |
| `vector::destroy_empty<T>(v: vector<T>)`                   | `v`を削除                                                                                                                                                      | `v`が空でない場合            |
| `vector::append<T>(v1: &mut vector<T>, v2: vector<T>)`     | `v2`の要素を`v1`の末尾に追加                                                                                                                     | なし                          |
| `vector::contains<T>(v: &vector<T>, e: &T): bool`          | `e`がベクター`v`に含まれている場合はtrueを返す。そうでなければfalseを返す                                                                                               | なし                          |
| `vector::swap<T>(v: &mut vector<T>, i: u64, j: u64)`       | ベクター`v`の`i`番目と`j`番目のインデックスの要素を交換                                                                                             | `i`または`j`が範囲外の場合 |
| `vector::reverse<T>(v: &mut vector<T>)`                    | ベクター`v`の要素の順序をその場で逆転                                                                                                   | なし                          |
| `vector::index_of<T>(v: &vector<T>, e: &T): (bool, u64)`   | `e`がベクター`v`のインデックス`i`にある場合は`(true, i)`を返す。そうでなければ`(false, 0)`を返す                                                                    | なし                          |
| `vector::remove<T>(v: &mut vector<T>, i: u64): T`          | ベクター`v`の`i`番目の要素を削除し、後続のすべての要素をシフト。これはO(n)で、ベクター内の要素の順序を保持                     | `i`が範囲外の場合        |
| `vector::swap_remove<T>(v: &mut vector<T>, i: u64): T`     | ベクター`v`の`i`番目の要素を最後の要素と交換してから要素をポップ。これはO(1)だが、ベクター内の要素の順序は保持しない | `i`が範囲外の場合        |

<!-- TODO 生成されたstdlibドキュメントにリンクすべき？多分？ -->

より多くの操作が時間とともに追加される可能性があります。

## 例

```move
use std::vector;

let mut v = vector::empty<u64>();
vector::push_back(&mut v, 5);
vector::push_back(&mut v, 6);

assert!(*vector::borrow(&v, 0) == 5, 42);
assert!(*vector::borrow(&v, 1) == 6, 42);
assert!(vector::pop_back(&mut v) == 6, 42);
assert!(vector::pop_back(&mut v) == 5, 42);
```

## `vector`の破棄とコピー

`vector<T>`の一部の動作は、要素型`T`のアビリティに依存します。例えば、`drop`を持たない要素を含むベクターは、上記の例の`v`のように暗黙的に破棄することはできません。それらは`vector::destroy_empty`で明示的に破棄する必要があります。

`vec`がゼロ個の要素を含まない限り、`vector::destroy_empty`は実行時にアボートすることに注意してください：

```move
fun destroy_any_vector<T>(vec: vector<T>) {
    vector::destroy_empty(vec) // deleting this line will cause a compiler error
}
```

しかし、`drop`を持つ要素を含むベクターをドロップする場合、エラーは発生しません：

```move
fun destroy_droppable_vector<T: drop>(vec: vector<T>) {
    // valid!
    // nothing needs to be done explicitly to destroy the vector
}
```

同様に、要素型が`copy`を持たない限り、ベクターはコピーできません。言い換えれば、`vector<T>`は`T`が`copy`を持つ場合にのみ`copy`を持ちます。必要に応じて暗黙的にコピーされることに注意してください：

```move
let x = vector[10];
let y = x; // implicit copy
let z = x;
(y, z)
```

大きなベクターのコピーは高価になる可能性があることを覚えておいてください。これが懸念事項である場合、`intended`使用法に注釈を付けることで、意図しないコピーを防ぐことができます。例えば：

```move
let x = vector[10];
let y = move x;
let z = x; // ERROR! x has been moved
(y, z)
```

詳細については、[型アビリティ](./../abilities)と[ジェネリクス](./../generics)のセクションを参照してください。

## 所有権

[上記](#destroying-and-copying-vectors)で述べたように、`vector`値は要素がコピーできる場合にのみコピーできます。その場合、コピーは[`copy`](./../variables#move-and-copy)または[逆参照`*`](./references#reading-and-writing-through-references)によって実行できます。
