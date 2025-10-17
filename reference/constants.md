---
title: '定数 | リファレンス'
description: ''
---

# 定数

定数は、`module`内で共有される静的な値に名前を付ける方法です。

定数の値はコンパイル時に既知である必要があります。定数の値はコンパイルされたモジュールに保存されます。そして定数が使用されるたびに、その値の新しいコピーが作成されます。

## 宣言

定数の宣言は`const`キーワードで始まり、続いて名前、型、値を指定します。

```text
const <name>: <type> = <expression>;
```

例えば

```move
module a::example;

const MY_ADDRESS: address = @a;

public fun permissioned(addr: address) {
    assert!(addr == MY_ADDRESS, 0);
}
```

## 命名

定数は大文字`A`から`Z`で始まる必要があります。最初の文字の後、定数名にはアンダースコア`_`、小文字`a`から`z`、大文字`A`から`Z`、または数字`0`から`9`を含めることができます。

```move
const FLAG: bool = false;
const EMyErrorCode: u64 = 0;
const ADDRESS_42: address = @0x42;
```

定数で小文字`a`から`z`を使用できますが、[一般的なスタイルガイドライン](./coding-conventions)では、大文字`A`から`Z`のみを使用し、各単語の間にアンダースコア`_`を入れることが推奨されています。エラーコードには、`E`を接頭辞として使用し、残りの名前にはアッパーキャメルケース（パスカルケースとも呼ばれます）を使用します。`EMyErrorCode`で確認できます。

現在の`A`から`Z`で始まる命名制約は、将来の言語機能のための余地を残すために設けられています。

## 可視性

`public`または`public(package)`定数は現在サポートされていません。`const`値は宣言したモジュール内でのみ使用できます。ただし、便宜上、[ユニットテスト属性](./unit-testing)ではモジュール間で使用できます。

## 有効な式

現在、定数はプリミティブ型`bool`、`u8`、`u16`、`u32`、`u64`、`u128`、`u256`、`address`、および`vector<T>`（`T`は定数に有効な型）に制限されています。

### 値

一般的に、`const`にはその型の単純な値またはリテラルが割り当てられます。例えば

```move
const MY_BOOL: bool = false;
const MY_ADDRESS: address = @0x70DD;
const BYTES: vector<u8> = b"hello world";
const HEX_BYTES: vector<u8> = x"DEADBEEF";
```

### 複雑な式

リテラルに加えて、コンパイラがコンパイル時に式を値に減らすことができる限り、定数にはより複雑な式を含めることができます。

現在、等価演算、すべてのブール演算、すべてのビット演算、およびすべての算術演算を使用できます。

```move
const RULE: bool = true && false;
const CAP: u64 = 10 * 100 + 1;
const SHIFTY: u8 = {
    (1 << 1) * (1 << 2) * (1 << 3) * (1 << 4)
};
const HALF_MAX: u128 = 340282366920938463463374607431768211455 / 2;
const REM: u256 =
    57896044618658097711785492504343953926634992332820282019728792003956564819968 % 654321;
const EQUAL: bool = 1 == 1;
```

演算が実行時例外を引き起こす場合、コンパイラは定数の値を生成できないというエラーを出します

```move
const DIV_BY_ZERO: u64 = 1 / 0; // ERROR!
const SHIFT_BY_A_LOT: u64 = 1 << 100; // ERROR!
const NEGATIVE_U64: u64 = 0 - 1; // ERROR!
```

さらに、定数は同じモジュール内の他の定数を参照することができます。

```move
const BASE: u8 = 4;
const SQUARE: u8 = BASE * BASE;
```

ただし、定数定義における循環はエラーを引き起こすことに注意してください。

```move
const A: u16 = B + 1;
const B: u16 = A + 1; // ERROR!
```
