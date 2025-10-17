# ストレージ関数

主要なストレージ操作を定義するモジュールは`sui::transfer`です。これは[Sui Framework](./../programmability/sui-framework)に依存するすべてのパッケージで暗黙的にインポートされるため、他の暗黙的にインポートされるモジュール（例：`std::option`や`std::vector`）と同様に、useステートメントを追加する必要はありません。

> クイックリファレンスとして、[付録C: 転送関数](./../appendix/transfer-functions.md)にはすべてのストレージ関数とオブジェクト状態のリストが含まれています。

## 概要

`transfer`モジュールは、各[所有権タイプ](./../object/ownership)のストレージ操作を実行するための関数を提供します。

1. [Transfer](#transfer) - オブジェクトをアドレスに送信し、_アドレス所有_状態にします；
2. [Freeze](#freeze) - オブジェクトを_不変_状態にし、_パブリック定数_となり、変更されることがなくなります。
3. [Share](#share) - オブジェクトを_共有_状態にし、誰でも利用できるようにします；

`transfer`モジュールは、[動的フィールド](./../programmability/dynamic-fields)の特別なケースを除いて、ほとんどのストレージ操作の定番です。これらは次の章で説明します。

## 所有権と参照：クイック復習

[所有権とスコープ](./../move-basics/ownership-and-scope)と[参照](./../move-basics/references)の章では、Moveでの所有権と参照の基本について説明しました。ストレージ関数を使用する際に、これらの概念を理解することが重要です。以下は最も重要なポイントのクイック復習です：

- Moveの_move_セマンティクスは、値が一つのスコープから別のスコープに_移動_されることを意味します。言い換えれば、型のインスタンスが関数に_値で_渡された場合、それは関数スコープに_移動_され、呼び出し元スコープではもうアクセスできません。
- 値の所有権を維持するには、_参照で_渡すことができます。_不変参照_`&T`または_可変参照_`&mut T`のいずれかです。その場合、値は_借用_され、呼び出し先スコープでアクセスできますが、所有者は同じままです。

```move
/// 値で移動
public fun take<T>(value: T) { /* 値はここに移動！ */ abort }

/// 不変参照の場合、値は親スコープに残る。
public fun borrow<T>(value: &T) { /* 値はここで借用！読み取り可能 */ abort }

/// 可変参照の場合、値は親スコープに残るが変更可能。
public fun borrow_mut<T>(value: &mut T) { /* 値はここで可変借用！ */ abort }
```

<!-- TODO part on:
    - object does not have an associated storage type
    - the same type of object can be stored differently
    - the objects must be specified in the transaction by their ID
 -->

## 転送関数での内部ルール

ストレージ操作はオブジェクトに対してのみ実行でき、_内部_と_パブリック_の2つの形式があります。内部、または時々_制限_と呼ばれる転送関数は、[`key`][key]のみの型に対して実行でき、名前の通り[内部制約](./internal-constraint.md)を強制します。パブリック版は`key`と[`store`][store]を持つ任意のオブジェクトで呼び出すことができます。したがって、`key`のみの型のストレージは完全にその定義モジュールによって管理され、`store`は他のモジュールでパブリック転送関数を呼び出すことを可能にします。

```move
/// T: 内部、`T`を定義するモジュールでのみ呼び出し可能。
public fun transfer<T: key>(obj: T, recipient: address);

/// `T`が呼び出し元に対して内部的である必要はないが、`store`を要求。
public fun public_transfer<T: key + store>(obj: T, recipient: address);
```

上記の例では、`transfer`関数は`T`を定義するモジュールからのみ呼び出すことができ、型制約`T: key`を持ちます。一方、`public_transfer`は名前で明確に示されているように、任意のモジュールから呼び出すことができますが、`T`が`key`と`store`を持つことを要求します。

このルールを知ることは、Moveでのアプリケーション設計を理解するために重要です。オブジェクトをパブリックに転送可能にする（`key`と`store`）か、内部に保つ（`key`のみ）かの選択は、アプリケーションロジックと今後の開発に劇的に影響する可能性があります。

## Transfer

`transfer::transfer`関数は、オブジェクトをアドレスに転送するために使用される関数です。そのシグネチャは以下の通りで、[`key`アビリティ](./key-ability.md)を持つ型と受信者の[アドレス](./../move-basics/address.md)のみを受け入れます。オブジェクトは関数に_値で_渡されるため、関数スコープに_移動_され、その後受信者アドレスに移動されることに注意してください。

```move
module sui::transfer;

// `obj`を`recipient`に転送。
public fun transfer<T: key>(obj: T, recipient: address);

// `transfer`関数のパブリック版。
public fun public_transfer<T: key + store>(obj: T, recipient: address);
```

### 転送の例

以下の例では、オブジェクトを定義してトランザクション送信者に送信するモジュールでどのように使用できるかを見ることができます。

```move
module book::transfer_to_sender;

/// `key`を持つ構造体はオブジェクトです。最初のフィールドは`id: UID`！
public struct AdminCap has key { id: UID }

/// `init`関数は、モジュールが公開されたときに呼び出される特別な関数です。
/// アプリケーションのセットアップを行うのに適した場所です。
fun init(ctx: &mut TxContext) {
    // このスコープで新しい`AdminCap`オブジェクトを作成。
    let admin_cap = AdminCap { id: object::new(ctx) };

    // オブジェクトをトランザクション送信者に転送。
    transfer::transfer(admin_cap, ctx.sender());
}

/// `AdminCap`オブジェクトを`recipient`に転送。したがって、受信者が
/// オブジェクトの所有者となり、彼らだけがアクセスできます。
public fun transfer_admin_cap(cap: AdminCap, recipient: address) {
    transfer::transfer(cap, recipient);
}
```

モジュールが公開されると、`init`関数が呼び出され、その中で作成した`AdminCap`オブジェクトがトランザクション送信者に_転送_されます。`ctx.sender()`関数は現在のトランザクションの送信者アドレスを返します。

`AdminCap`が送信者（例えば`0xa11ce`）に転送されると、送信者、そして送信者のみがオブジェクトにアクセスできるようになります。このタイプの所有権は_アドレス所有権_と呼ばれます。

> アドレス所有オブジェクトは_真の所有権_の対象です - 所有者アドレスのみがアクセスできます。これはSuiストレージモデルの基本的な概念です。

### パブリック転送

`AdminCap`を使用して新しいオブジェクトのミントとアドレスへの転送を認証する関数で例を拡張してみましょう：

```move
/// 管理者がアドレスに`mint_and_transfer`できる`Gift`オブジェクト。
public struct Gift has key, store { id: UID }

/// 新しい`Gift`オブジェクトを作成し、`recipient`に転送。
public fun mint_and_transfer(
    _: &AdminCap, recipient: address, ctx: &mut TxContext
) {
    let gift = Gift { id: object::new(ctx) };
    transfer::public_transfer(gift, recipient);
}
```

`mint_and_transfer`関数は、誰でも呼び出す「可能性がある」_パブリック_関数ですが、最初の引数として`AdminCap`への参照を必要とします。それがないと、関数は呼び出し可能ではありません。これは、特権関数へのアクセスを制限するシンプルで非常に明示的な方法で、_[Capability](./../programmability/capability)_と呼ばれます。`AdminCap`オブジェクトが_アドレス所有_であるため、`0xa11ce`のみが`mint_and_transfer`関数を呼び出すことができます。

`AdminCap`では`key`アビリティのみを追加して転送可能性と使用可能性の両方を制限しましたが、`Gift`は`key`と`store`の組み合わせを持っているため、`Gift`を所有する人は誰でも`transfer::public_transfer`を自由に呼び出して他の人に送ることができます。`store`がない場合、現在の実装では`Gift`は_"ソウルバウンド"_（魂に縛られた）となり、`Gift`の幸せな所有者はそれで何もできなくなってしまいます。

### クイック復習

- `transfer`関数はオブジェクトをアドレスに送信するために使用される；
- オブジェクトは_アドレス所有_となり、受信者のみがアクセスできる；
- _アドレス所有_オブジェクトは参照または値で使用でき、別のアドレスに転送することも可能；
- その_パブリック_版は`public_transfer`で、`store`を要求する；
- 関数は、オブジェクトを引数として渡すことを要求することで制限され、_capability_を作成する。

## Freeze

`transfer::freeze_object`関数は、オブジェクトを_不変_状態にするために使用される関数です。オブジェクトが_凍結_されると、二度と変更されることがなく、誰でも不変参照でアクセスできるようになります。

関数シグネチャは以下の通りで、[`key`アビリティ](./key-ability)を持つ型のみを受け入れます。他のすべてのストレージ関数と同様に、オブジェクトを_値で_受け取ります。この関数のパブリック版は`public_freeze_object`で、`T`が`store`を持つことを要求します。

```move
module sui::transfer;

// オブジェクトを不変にし、誰でも読み取り可能にする。
public fun freeze_object<T: key>(obj: T);

// `freeze_object`関数のパブリック版。
public fun public_freeze_object<T: key + store>(obj: T);
```

前の例を拡張して、管理者が`Config`オブジェクトを作成して凍結することを可能にする関数を追加しましょう：

```move
/// 管理者が`create_and_freeze`できる`Config`オブジェクト。
public struct Config has key {
    id: UID,
    message: String
}

/// 新しい`Config`オブジェクトを作成し、凍結。
public fun create_and_freeze(
    _: &AdminCap,
    message: String,
    ctx: &mut TxContext
) {
    let config = Config {
        id: object::new(ctx),
        message
    };

    // オブジェクトを凍結して不変にする。
    transfer::freeze_object(config);
}

/// `Config`オブジェクトからメッセージを返す。
/// 不変参照でオブジェクトにアクセス可能！
public fun message(c: &Config): String { c.message }
```

Configは`message`フィールドを持つオブジェクトで、`create_and_freeze`関数は新しい`Config`を作成して凍結します。オブジェクトが凍結されると、誰でも不変参照でアクセスできるようになります。`message`関数は`Config`オブジェクトからメッセージを返すパブリック関数です。Configは今やIDでパブリックに利用可能で、メッセージは誰でも読み取ることができます。

> 関数定義はオブジェクトの状態と関連していません。常に凍結されている型に対して可変参照を取る関数を定義することは可能です。しかし、凍結されたオブジェクトでは呼び出し可能ではありません。

上記の例の`message`関数は、不変の`Config`オブジェクトで呼び出すことができます。しかし、以下に示す2つの関数は凍結されたオブジェクトでは呼び出し可能ではありません：

```move
// === これらは凍結されたオブジェクトでは呼び出せません！ ===

/// 関数は定義できるが、凍結されたオブジェクトでは呼び出し可能ではない。
/// 不変参照のみが許可される。
public fun message_mut(c: &mut Config): &mut String { &mut c.message }

/// `Config`オブジェクトを削除、値で受け取る。
/// 凍結されたオブジェクトでは呼び出せません！
public fun delete_config(c: Config) {
    let Config { id, message: _ } = c;
    id.delete()
}
```

まとめると：

- `transfer::freeze_object`関数はオブジェクトを_不変_状態にするために使用される；
- オブジェクトが_凍結_されると、二度と変更、削除、転送されることがなく、誰でも不変参照でアクセスできる；
- `freeze_object`関数の_パブリック_版は`public_freeze_object`で、`T`が`store`を持つことを要求する。

## 所有 -> 凍結

`transfer::freeze_object`シグネチャは`key`アビリティを持つ任意の型を受け入れるため、同じスコープで作成されたオブジェクトを取ることもできますが、アカウントによって所有されていたオブジェクトを取ることもできます。これは、`freeze_object`関数が_転送_されたオブジェクトを_凍結_するために使用できることを意味します。セキュリティ上の懸念から、`AdminCap`オブジェクトを凍結したくはありません - 誰でもアクセスできるようになるため、セキュリティリスクになります。しかし、ミントされて受信者に転送された`Gift`オブジェクトを凍結することはできます：

> 単一所有者 -> 不変変換が可能！

```move
/// `Gift`オブジェクトを凍結して不変にする。
/// Giftは`key`と`store`を持つため、`public_freeze_object`を使用可能！
public fun freeze_gift(gift: Gift) {
    transfer::public_freeze_object(gift);
}
```

## Share

`transfer::share_object`関数は、オブジェクトを_共有_状態にするために使用される関数です。オブジェクトが_共有_されると、誰でも可変参照（したがって、不変参照も）でアクセスできるようになります。関数シグネチャは以下の通りで、[`key`アビリティ](./key-ability)を持つ型のみを受け入れます：

```move
module sui::transfer;

/// オブジェクトを共有状態にする - 可変および不変でアクセス可能。
public fun share_object<T: key>(obj: T);

/// `share_object`関数のパブリック版。
public fun public_share_object<T: key + store>(obj: T);
```

他の転送関数と同様に、`share_object`は`T`が`store`を持つことを要求する_パブリック_版を持ちます。

オブジェクトが_共有_されると、可変参照としてパブリックに利用可能になります。

## 特別なケース：共有オブジェクトの削除

共有オブジェクトは通常値で取ることはできませんが、それを取る関数がオブジェクトを削除する場合という特別なケースがあります。これはSuiストレージモデルの特別なケースで、共有オブジェクトの削除を可能にするために使用されます。これがどのように動作するかを示すために、Configオブジェクトを作成して共有し、その後削除する関数を作成します：

```move
/// 新しい`Config`オブジェクトを作成し、共有。
public fun create_and_share(message: String, ctx: &mut TxContext) {
    let config = Config {
        id: object::new(ctx),
        message
    };

    // オブジェクトを共有して共有状態にする。
    transfer::share_object(config);
}
```

`create_and_share`関数は新しい`Config`オブジェクトを作成して共有します。オブジェクトは今や可変参照としてパブリックに利用可能です。共有オブジェクトを削除する関数を作成しましょう：

```move
/// `Config`オブジェクトを削除、値で受け取る。
/// 共有オブジェクトで呼び出し可能！
public fun delete_config(c: Config) {
    let Config { id, message: _ } = c;
    id.delete()
}
```

`delete_config`関数は`Config`オブジェクトを値で受け取って削除し、Sui Verifierはこの呼び出しを許可します。しかし、関数が`Config`オブジェクトを返すか、`freeze`や`transfer`しようとすると、Sui Verifierはトランザクションを拒否します。

```move
// 動作しません！
public fun transfer_shared(c: Config, to: address) {
    transfer::transfer(c, to);
}
```

まとめると：

- `share_object`関数はオブジェクトを_共有_状態にするために使用される；
- オブジェクトが_共有_されると、誰でも可変参照でアクセスできる；
- 共有オブジェクトは削除できるが、転送や凍結はできない；
- `share_object`関数の_パブリック_版は`public_share_object`で、`T`が`store`を持つことを要求する。

## 次のステップ

`transfer`モジュールの主要な機能を知ったので、ストレージ操作を含むより複雑なSuiアプリケーションの構築を開始できます。次の章では、オブジェクト内にデータを保存し、ここでほとんど触れなかった転送制限を緩和する[Storeアビリティ](./store-ability)について説明します。その後、Suiストレージモデルで最も重要な型である[UIDとID](./uid-and-id)について説明します。

[key]: ./key-ability.md
[store]: ./store-ability.md