# Sui Verifier: 内部制約

Suiバイトコード検証器は、重要なストレージ操作の安全性を確保するために、Moveバイトコードに対して一連のルールを強制します。これらのルールの一つが_内部制約_です。これは、型パラメータ`T`を持つ関数の呼び出し元が、その型の_定義モジュール_でなければならないことを要求します。言い換えれば、Tは呼び出しを行うモジュールに対して_内部的_でなければなりません。

このルールは（まだ）Move言語自体の一部ではないため、不透明に感じられるかもしれません。それでも、特にSuiでのストレージ関連操作を扱う際には、理解すべき重要なルールです。

[Sui Framework][sui-framework]の例を見てみましょう。[`sui::event`][event]モジュールのemit関数は、その型パラメータ`T`が呼び出し元に対して_内部的_であることを要求します：

```move
// `T`に`internal`を強制する関数の実際の例。
module sui::event;

// この関数が`T`を定義しないモジュールから呼び出された場合、
// Sui Verifierはコンパイル時にエラーを出力します。
public native fun emit<T: copy + drop>(event: T);
```

以下は`emit`の正しい呼び出しです。型`A`は`exercise_internal`モジュール内で定義されているため、内部的で有効です：

```move
// 型`A`を定義。
module book::exercise_internal;

use sui::event;

/// このモジュールで定義された型なので、ここでは内部的です。
public struct A has copy, drop {}

// `A`がローカルで定義されているため、これは動作します。
public fun call_internal() {
    event::emit(A {})
}
```

しかし、他の場所で定義された型で`emit`を呼び出そうとすると、検証器はそれを拒否します。例えば、この関数を同じモジュールに追加すると、[Standard Library][move-stdlib]の`TypeName`型を使用しようとするため失敗します：

```move
// これは失敗します！
public fun call_foreign_fail() {
    use std::type_name;

    event::emit(type_name::get<A>());
    // ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ 無効なイベント。
    // エラー: `sui::event::emit`は現在のモジュールで定義された型で
    // 呼び出される必要があります。
}
```

内部制約は[Sui Framework][sui-framework]の特定の関数にのみ適用されます。この概念については、本書を通じて何度か言及します。

[sui-framework]: ./../programmability/sui-framework.md
[move-stdlib]: ./../move-basics/standard-library.md
[event]: ./../programmability/events.md
[reflection]: ./../move-basics/type-reflection.md