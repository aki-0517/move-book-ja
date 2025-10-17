---
title: '等価性 | リファレンス'
description: ''
---

# 等価性

Moveは2つの等価演算子`==`と`!=`をサポートします

## 操作

| 構文 | 操作 | 説明                                                                 |
| ------ | --------- | --------------------------------------------------------------------------- |
| `==`   | 等価     | 2つのオペランドが同じ値を持つ場合`true`を返し、そうでなければ`false`を返します   |
| `!=`   | 非等価 | 2つのオペランドが異なる値を持つ場合`true`を返し、そうでなければ`false`を返します |

### 型付け

等価（`==`）と非等価（`!=`）の両方の操作は、両方のオペランドが同じ型である場合にのみ機能します

```move
0 == 0; // `true`
1u128 == 2u128; // `false`
b"hello" != x"00"; // `true`
```

等価性と非等価性は、_すべて_のユーザー定義型でも機能します！

```move
module 0::example;

public struct S has copy, drop { f: u64, s: vector<u8> }

fun always_true(): bool {
    let s = S { f: 0, s: b"" };
    s == s
}

fun always_false(): bool {
    let s = S { f: 0, s: b"" };
    s != s
}
```

オペランドが異なる型を持つ場合、型チェックエラーが発生します

```move
1u8 == 1u128; // ERROR!
//     ^^^^^ expected an argument of type 'u8'
b"" != 0; // ERROR!
//     ^ expected an argument of type 'vector<u8>'
```

### 参照での型付け

[参照](./primitive-types/references)を比較する場合、参照の型（不変または可変）は関係ありません。これは、不変`&`参照を同じ基本型の可変`&mut`参照と比較できることを意味します。

```move
let i = &0;
let m = &mut 1;

i == m; // `false`
m == i; // `false`
m == m; // `true`
i == i; // `true`
```

上記は、必要に応じて各可変参照に明示的なfreezeを適用することと同等です

```move
let i = &0;
let m = &mut 1;

i == freeze(m); // `false`
freeze(m) == i; // `false`
m == m; // `true`
i == i; // `true`
```

ただし、基本型は同じ型である必要があります

```move
let i = &0;
let s = &b"";

i == s; // ERROR!
//   ^ expected an argument of type '&u64'
```

### 自動借用

Move 2024エディションから、`==`と`!=`演算子は、オペランドの1つが参照で他方がそうでない場合、オペランドを自動的に借用します。これは、以下のコードがエラーなしで動作することを意味します：

```move
let r = &0;

// すべての場合で、`0`は`&0`として自動的に借用されます
r == 0; // `true`
0 == r; // `true`
r != 0; // `false`
0 != r; // `false`
```

この自動借用は常に不変借用です。

## 制限

`==`と`!=`の両方は、比較する際に値を消費します。その結果、型システムは型が[`drop`](./abilities)を持つことを強制します。[`drop`アビリティ](./abilities)なしでは、所有権は関数の終了までに転送される必要があり、そのような値は宣言モジュール内でのみ明示的に破棄できることを思い出してください。これらが等価`==`または非等価`!=`で直接使用された場合、値が破棄され、[`drop`アビリティ](./abilities)の安全性保証が破られることになります！

```move
module 0::example;

public struct Coin has store { value: u64 }
fun invalid(c1: Coin, c2: Coin) {
    c1 == c2 // ERROR!
//  ^^    ^^ これらのアセットは破棄されるでしょう！
}
```

しかし、プログラマーは値を直接比較する代わりに、常に値を最初に借用することができ、参照型は[`drop`アビリティ](./abilities)を持ちます。例えば

```move
module 0::example;

public struct Coin has store { value: u64 }
fun swap_if_equal(c1: Coin, c2: Coin): (Coin, Coin) {
    let are_equal = &c1 == c2; // 有効、`c2`が自動的に借用されることに注意
    if (are_equal) (c2, c1) else (c1, c2)
}
```

## 余分なコピーを避ける

プログラマーは[`drop`](./abilities)を持つ型の任意の値を比較_できます_が、プログラマーは高価なコピーを避けるために参照で比較することがよくあります。

```move
let v1: vector<u8> = function_that_returns_vector();
let v2: vector<u8> = function_that_returns_vector();
assert!(copy v1 == copy v2, 42);
//      ^^^^       ^^^^
use_two_vectors(v1, v2);

let s1: Foo = function_that_returns_large_struct();
let s2: Foo = function_that_returns_large_struct();
assert!(copy s1 == copy s2, 42);
//      ^^^^       ^^^^
use_two_foos(s1, s2);
```

このコードは完全に受け入れ可能ですが（`Foo`が[`drop`](./abilities)を持つと仮定）、効率的ではありません。ハイライトされたコピーは削除して借用に置き換えることができます

```move
let v1: vector<u8> = function_that_returns_vector();
let v2: vector<u8> = function_that_returns_vector();
assert!(&v1 == &v2, 42);
//      ^      ^
use_two_vectors(v1, v2);

let s1: Foo = function_that_returns_large_struct();
let s2: Foo = function_that_returns_large_struct();
assert!(&s1 == &s2, 42);
//      ^      ^
use_two_foos(s1, s2);
```

`==`自体の効率は同じままですが、`copy`が削除されるため、プログラムはより効率的になります。
