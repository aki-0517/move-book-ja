---
draft: true
---

<!-- このページは非推奨です。今は内容を保存していますが、storage-functionsにリダイレクトすべきです -->

# 制限付きとパブリック転送

[前のセクション](./storage-functions)で説明したストレージ操作は、デフォルトで制限されています - オブジェクトを定義するモジュールでのみ呼び出すことができます。言い換えれば、型はストレージ操作で使用されるために、モジュールに対して_内部的_でなければなりません。この制限はSui Verifierに実装されており、バイトコードレベルで強制されます。

しかし、オブジェクトを他のモジュールで転送・保存できるようにするために、これらの制限を緩和することができます。`sui::transfer`モジュールは、他のモジュールでストレージ操作を呼び出すことを可能にする一連の_public\_\*_関数を提供します。これらの関数は`public_`で始まり、すべてのモジュールとトランザクションで利用可能です。

## パブリックストレージ操作

`sui::transfer`モジュールは以下のパブリック関数を提供します。これらは既に説明したものとほぼ同じですが、任意のモジュールから呼び出すことができます。

```move
module sui::transfer;

/// `transfer`関数のパブリック版。
public fun public_transfer<T: key + store>(object: T, to: address) {}

/// `share_object`関数のパブリック版。
public fun public_share_object<T: key + store>(object: T) {}

/// `freeze_object`関数のパブリック版。
public fun public_freeze_object<T: key + store>(object: T) {}
```

これらの関数の使用法を説明するために、以下の例を考えてみましょう：モジュールAが`key`を持つObjectKと`key + store`を持つObjectKSを定義し、モジュールBがこれらのオブジェクトの`transfer`関数を実装しようとします。

> この例では`transfer::transfer`を使用していますが、`share_object`と`freeze_object`関数でも同じ動作をします。

```move
/// `key`と`key + store`アビリティを持つ`ObjectK`と`ObjectKS`を定義
module book::transfer_a;

public struct ObjectK has key { id: UID }
public struct ObjectKS has key, store { id: UID }
```

```move
/// `transfer_a`から`ObjectK`と`ObjectKS`型をインポートし、
/// それらの異なる`transfer`関数を実装しようとする
module book::transfer_b;

// 型はこのモジュールに対して内部的ではない
use book::transfer_a::{ObjectK, ObjectKS};

// 失敗！ObjectKは`store`ではなく、ObjectKはこのモジュールに対して内部的ではない
public fun transfer_k(k: ObjectK, to: address) {
    transfer::transfer(k, to);
}

// 失敗！ObjectKSは`store`を持つが、関数はパブリックではない
public fun transfer_ks(ks: ObjectKS, to: address) {
    transfer::transfer(ks, to);
}

// 失敗！ObjectKは`store`ではなく、`public_transfer`は`store`を要求する
public fun public_transfer_k(k: ObjectK, to: address) {
    transfer::public_transfer(k, to);
}

// 動作する！ObjectKSは`store`を持ち、関数はパブリック
public fun public_transfer_ks(ks: ObjectKS, to: address) {
    transfer::public_transfer(ks, to);
}
```

上記の例を拡張すると：

- ❌ `transfer_k`は失敗 - ObjectKはモジュール`transfer_b`に対して内部的ではない
- ❌ `transfer_ks`は失敗 - ObjectKSはモジュール`transfer_b`に対して内部的ではない
- ❌ `public_transfer_k`は失敗 - ObjectKは`store`アビリティを持たない
- ✅ `public_transfer_ks`は動作 - ObjectKSは`store`アビリティを持ち、転送はパブリック

## `store`の影響

型に`store`アビリティを追加するかどうかの決定は慎重に行うべきです。一方では、型が他のアプリケーションで_使用可能_であるための事実上の要件です。他方では、_ラッピング_と意図されたストレージモデルの変更を可能にします。例えば、キャラクターはアカウントによって所有されることを意図しているかもしれませんが、`store`アビリティがあると、凍結される可能性があります（共有はできません - この遷移は制限されています）。