# パターン：ウィットネス

ウィットネスは、証明を構築することによって存在を証明するパターンです。プログラミングの文脈では、
ウィットネスは、プロパティが保持されている場合にのみ構築できる値を提供することによって、
システムの特定のプロパティを証明する方法です。

## Moveでのウィットネス

[Struct](./../move-basics/struct)セクションでは、構造体はそれを定義するモジュールによってのみ
作成（または_パック_）できることを示しました。したがって、Moveでは、モジュールは型を構築することによって
その型の所有権を証明します。これはMoveで最も重要なパターンの一つであり、
ジェネリック型のインスタンス化と認証に広く使用されています。

実際には、ウィットネスを使用するには、ウィットネスを引数として期待する関数が
必要です。以下の例では、`Instance<T>`インスタンスを作成するために`T`型のウィットネスを
期待する`new`関数です。

> ウィットネス構造体が格納されないことが多く、そのため関数は型に
> [Drop](./../move-basics/drop-ability)アビリティを要求する場合があります。

```move
module book::witness;

/// A struct that requires a witness to be created.
public struct Instance<T> { t: T }

/// Create a new instance of `Instance<T>` with the provided T.
public fun new<T>(witness: T): Instance<T> {
    Instance { t: witness }
}
```

`Instance<T>`を構築する唯一の方法は、型`T`のインスタンスで`new`関数を呼び出すことです。
これはMoveでのウィットネスパターンの基本的な例です。ウィットネスを提供するモジュールは、
以下の`book::witness_source`モジュールのように、対応する実装を持つことがよくあります：

```move
module book::witness_source;

use book::witness::{Self, Instance};

/// ウィットネスとして使用される構造体。
public struct W {}

/// `Instance<W>`の新しいインスタンスを作成します。
public fun new_instance(): Instance<W> {
    witness::new(W {})
}
```

構造体`W`のインスタンスは`new_instance`関数に渡されて`Instance<W>`を作成し、
それによって`book::witness_source`モジュールが型`W`を所有していることを証明します。

## ジェネリック型のインスタンス化

ウィットネスにより、ジェネリック型を具体的な型でインスタンス化できます。
これは、モジュールがその能力を提供する場合、型から関連する動作を継承し、
それらを拡張するオプションを持つのに役立ちます。

```move
module sui::balance;

/// A Supply of T. Used for minting and burning.
public struct Supply<phantom T> has store {
    value: u64,
}

/// Create a new supply for type T with the provided witness.
public fun create_supply<T: drop>(_w: T): Supply<T> {
    Supply { value: 0 }
}

/// Get the `Supply` value.
public fun supply_value<T>(supply: &Supply<T>): u64 {
    supply.value
}
```

上記の例は[Sui Framework](./sui-framework)の[`balance` module][balance-framework]から
借用したもので、`Supply`は型`T`のウィットネスを提供することによってのみ構築できる
ジェネリック構造体です。ウィットネスは値で受け取られ、_破棄_されます - したがって`T`は
[drop](./../move-basics/drop-ability)アビリティを持たなければなりません。

[balance-framework]: https://docs.sui.io/references/framework/sui/balance

インスタンス化された`Supply<T>`は、`T`が供給の型である新しい`Balance<T>`を
ミントするために使用できます。

```move
module sui::balance;

const EOverflow: u64 = 0;

/// 格納可能なバランス。
public struct Balance<phantom T> has store {
    value: u64,
}

/// 供給を`value`だけ増やし、この値で新しい`Balance<T>`を作成します。
public fun increase_supply<T>(self: &mut Supply<T>, value: u64): Balance<T> {
    assert!(value < (std::u64::max_value!() - self.value), EOverflow);
    self.value = self.value + value;
    Balance { value }
}
```

## ワンタイムウィットネス

構造体は任意の回数作成できますが、構造体が一度だけ作成されることを保証する必要がある
場合があります。この目的のために、Suiは「ワンタイムウィットネス」を提供します - 
一度だけ使用できる特別なウィットネスです。これについては[次のセクション](./one-time-witness)で
より詳しく説明します。

## まとめ

- ウィットネスは、証明を構築することによって特定のプロパティを証明するパターンです。
- Moveでは、モジュールは型を構築することによってその型の所有権を証明します。
- ウィットネスは、ジェネリック型のインスタンス化と認証によく使用されます。

## 次のステップ

次のセクションでは、[ワンタイムウィットネス](./one-time-witness)パターンについて学びます。
