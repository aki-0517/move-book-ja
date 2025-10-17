# 文字列

Moveには文字列を表現する組み込み型はありませんが、[標準ライブラリ](./standard-library)に文字列の2つの標準実装があります。`std::string`モジュールはUTF-8エンコード文字列の`String`型とメソッドを定義し、2番目のモジュール`std::ascii`はASCII `String`型とそのメソッドを提供します。

> Sui実行環境は、トランザクション入力でバイトベクターを自動的に`String`に変換します。その結果、多くの場合、[Transaction Block](./../concepts/what-is-a-transaction)内で直接Stringを構築する必要はありません。

<!--

## Bytestring Literal

TODO:

- reference vector
- reference literals - [Expression](./expression#literals)

-->

## 文字列はバイト

どの種類の文字列を使用しても、文字列は単なるバイトであることを知っておくことが重要です。`string`と`ascii`モジュールによって提供されるラッパーは、まさにそのとおりラッパーです。文字列を扱うための安全性チェックとメソッドを提供しますが、結局のところ、それらは単なるバイトのベクターです。

```move file=packages/samples/sources/move-basics/string.move anchor=custom

```

## UTF-8文字列の操作

標準ライブラリには2つの種類の文字列（`string`と`ascii`）がありますが、`string`モジュールをデフォルトと考えるべきです。多くの一般的な操作のネイティブ実装を持ち、低レベルで最適化されたランタイムコードを活用して優れたパフォーマンスを実現します。対照的に、`ascii`モジュールは完全にMoveで実装されており、高レベルの抽象化に依存しているため、パフォーマンスクリティカルなタスクには適していません。

### 定義

`std::string`モジュールの`String`型は以下のように定義されています：

```move
module std::string;

/// `String`はutf8形式であることが保証されたバイトシーケンスを保持します。
public struct String has copy, drop, store {
    bytes: vector<u8>,
}
```

_[std::string][string-stdlib]モジュールの完全なドキュメント参照。_

### 文字列の作成

新しいUTF-8 `String`インスタンスを作成するには、`string::utf8`メソッドを使用できます。[標準ライブラリ](./standard-library)は便利のため`vector<u8>`にエイリアス`.to_string()`を提供しています。

```move file=packages/samples/sources/move-basics/string.move anchor=utf8

```

### 一般的な操作

UTF8 Stringは文字列を扱うための多くのメソッドを提供します。文字列の最も一般的な操作は、連結、スライス、長さの取得です。さらに、カスタム文字列操作のために、`bytes()`メソッドを使用して基になるバイトベクターを取得できます。

```move
let mut str = b"Hello,".to_string();
let another = b" World!".to_string();

// append(String)は文字列の末尾にコンテンツを追加します
str.append(another);

// `sub_string(start, end)`は文字列のスライスをコピーします
str.sub_string(0, 5); // "Hello"

// `length()`は文字列のバイト数を返します
str.length(); // 12 (bytes)

// メソッドはチェーンもできます！サブストリングの長さを取得
str.sub_string(0, 5).length(); // 5 (bytes)

// 文字列が空かどうか
str.is_empty(); // false

// カスタム操作のために基になるバイトベクターを取得
let bytes: &vector<u8> = str.bytes();
```

### 安全なUTF-8操作

デフォルトの`utf8`メソッドは、渡されたバイトが有効なUTF-8でない場合にアボートする可能性があります。渡すバイトが有効かどうか確信がない場合は、代わりに`try_utf8`メソッドを使用すべきです。これは`Option<String>`を返し、バイトが有効なUTF-8でない場合は値を含まず、そうでなければ文字列を含みます。

> ヒント：`try_*`で始まる名前の関数は通常`Option`を返します。操作が成功すると、結果は`Some`でラップされます。失敗すると、関数は`None`を返します。この命名規則はMoveで一般的に使用され、Rustからインスパイアされています。

```move file=packages/samples/sources/move-basics/string.move anchor=safe_utf8

```

### UTF-8の制限

`string`モジュールは文字列内の個別の文字にアクセスする方法を提供しません。これは、UTF-8が可変長エンコーディングであり、文字の長さが1から4バイトまでの範囲になる可能性があるためです。同様に、`length()`メソッドは文字数ではなく、文字列内のバイト数を返します。

ただし、`sub_string`や`insert`などのメソッドは文字境界を検証し、指定されたインデックスが文字の途中にある場合はアボートします。

## ASCII文字列

このセクションは近日公開予定です！

## 参考文献

- [std::string][string-stdlib]モジュールドキュメント。
- [std::ascii][ascii-stdlib]モジュールドキュメント。

[enum-reference]: /reference/enums.html
[string-stdlib]: https://docs.sui.io/references/framework/std/string
[ascii-stdlib]: https://docs.sui.io/references/framework/std/ascii
