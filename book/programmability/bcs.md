# バイナリ正規シリアライゼーション

バイナリ正規シリアライゼーション（BCS）は、構造化データのためのバイナリエンコーディング形式です。
これは元々Diemで設計され、Moveの標準シリアライゼーション形式になりました。BCSはシンプルで、
効率的、決定的、そして任意のプログラミング言語で実装しやすいものです。

> 完全な形式仕様は[BCSリポジトリ](https://github.com/zefchain/bcs)で利用可能です。

## 形式

BCSは、256ビットまでの符号なし整数、オプション、ブール値、ユニット（空の値）、
固定長および可変長シーケンス、マップをサポートするバイナリ形式です。この形式は
決定的になるように設計されており、同じデータは常に同じバイトにシリアライゼーションされます。

> 「BCSは自己記述形式ではありません。そのため、メッセージをデシリアライゼーションするには、
> 事前にメッセージタイプとレイアウトを知っている必要があります」[README](https://github.com/zefchain/bcs)より

整数はリトルエンディアン形式で格納され、可変長整数は可変長エンコーディングスキームを使用して
エンコードされます。シーケンスは長さをULEB128としてプレフィックスし、列挙型は
バリアントのインデックスに続いてデータを格納し、マップはキーと値のペアの順序付きシーケンスとして
格納されます。

構造体はフィールドのシーケンスとして扱われ、フィールドは構造体で定義された順序で
シリアライゼーションされます。フィールドはトップレベルデータと同じルールを使用して
シリアライゼーションされます。

## BCSの使用

[Sui Framework](./sui-framework)には、データのエンコードとデコードのための[`sui::bcs`][sui-bcs]モジュールが
含まれています。エンコード関数はVMネイティブであり、デコード関数はMoveで実装されています。

## エンコード

データをエンコードするには、データ参照をバイトベクターに変換する`bcs::to_bytes`関数を使用します。
この関数は、構造体を含む任意の型のエンコードをサポートします。

```move
module std::bcs;

public native fun to_bytes<T>(t: &T): vector<u8>;
```

以下の例は、BCSを使用して構造体をエンコードする方法を示しています。`to_bytes`関数は任意の
値を取り、それをバイトのベクターとしてエンコードできます。

```move file=packages/samples/sources/programmability/bcs.move anchor=encode

```

### 構造体のエンコード

構造体は単純な型と同様にエンコードされます。以下は、BCSを使用して構造体をエンコードする方法です：

```move file=packages/samples/sources/programmability/bcs.move anchor=encode_struct

```

## デコード

BCSは自己記述形式ではないため、デコードにはデータ型の事前知識が必要です。
[`sui::bcs`][sui-bcs]モジュールは、このプロセスを支援するための様々な関数を提供します。

### ラッパーAPI

BCSはMoveでラッパーとして実装されています。デコーダーはバイトを値で受け取り、
`peel_*`でプレフィックスされた異なるデコード関数を呼び出すことで、呼び出し元がデータを
「剥がす」ことを可能にします。データはバイトから抽出され、`into_remainder_bytes`関数が
呼び出されるまで、残りのバイトはラッパーに保持されます。

```move file=packages/samples/sources/programmability/bcs.move anchor=decode

```

デコード中に単一の`let`文で複数の変数を使用するのは一般的な実践です。これにより
コードが少し読みやすくなり、データの不要なコピーを避けるのに役立ちます。

```move file=packages/samples/sources/programmability/bcs.move anchor=chain_decode

```

### ベクターのデコード

ほとんどのプリミティブ型には専用のデコード関数がありますが、ベクターは要素の型に依存する
特別な処理が必要です。ベクターの場合、まずベクターの長さをデコードし、
次にループで各要素をデコードする必要があります。

```move file=packages/samples/sources/programmability/bcs.move anchor=decode_vector

```

この機能は、ライブラリによって`peel_vec!`マクロとして提供されています。これは
ベクターの長さと同じ回数だけ内部式を呼び出し、結果を単一のベクターに集約します。

```move
let u64_vec = bcs.peel_vec!(|bcs| bcs.peel_u64());
let address_vec = bcs.peel_vec!(|bcs| bcs.peel_address());

// 注意：これは`MyStruct`が現在のモジュールで定義されている場合のみ可能です！
let my_struct = bcs.peel_vec!(|bcs| MyStruct {
    user_addr: bcs.peel_address(),
    age: bcs.peel_u8(),
});
```

### Optionのデコード

<!--
> 偶然にも、MoveのOptionはベクターであるため、BCSの単一バリアントを持つ列挙型の表現と
> 重複し、RustのOptionをMoveのものと完全に互換性を持たせています。
-->

Moveの[Option](./../move-basics/option)は、0または1の要素を持つベクターとして表現されます。
オプションを読み取るには、それをベクターのように扱い、その長さ（最初のバイト - 1または0）を
チェックします。

```move file=packages/samples/sources/programmability/bcs.move anchor=decode_option

```

[ベクター](#decoding-vectors)と同様に、バリアントインデックスをチェックし、
基になる値が_some_の場合に式を評価するラッパーマクロ`peel_option!`があります。

```move
let u8_opt = bcs.peel_option!(|bcs| bcs.peel_u8());
let bool_opt = bcs.peel_option!(|bcs| bcs.peel_bool());
```

### 構造体のデコード

構造体はフィールドごとにデコードされ、バイトをMove構造体に自動的にデコードする方法はありません。
バイトを構造体にパースするには、各フィールドをデコードし、型をインスタンス化する必要があります。

```move file=packages/samples/sources/programmability/bcs.move anchor=decode_struct

```

## まとめ

バイナリ正規シリアライゼーションは、構造化データのための効率的なバイナリ形式であり、
プラットフォーム間での一貫したシリアライゼーションを保証します。Sui Frameworkは
BCSを扱うための包括的なツールを提供し、組み込み関数を通じて広範な機能を可能にします。

[sui-bcs]: https://docs.sui.io/references/framework/sui_sui/bcs
