# ワンタイムウィットネス

通常の[Witness](./witness-pattern)は型の所有権を静的に証明する優れた方法ですが、
Witnessが一度だけインスタンス化されることを保証する必要がある場合があります。
これがワンタイムウィットネス（OTW）の目的です。

<!--
自己へのメモ：
  - 背景を最初にするか定義を最初にするか - どちらが良いか？
  - なぜ誰かがこのセクションを読むのか？
  - ドキュメントからOTWを削除した場合、定義を最初に与えるべき。
-->

## 定義

OTWは一度だけ使用できる特別なタイプのWitnessです。手動で作成することはできず、
モジュールごとに一意であることが保証されています。Sui Adapterは、以下のルールに従う型を
OTWとして扱います：

1. `drop`アビリティのみを持つ。
2. フィールドを持たない。
3. ジェネリック型ではない。
4. モジュール名をすべて大文字で命名する。

以下がOTWの例です：

```move file=packages/samples/sources/programmability/one-time-witness.move anchor=definition

```

OTWは手動で構築することはできず、そうしようとするコードはコンパイルエラーになります。
OTWは[モジュールイニシャライザー](./module-initializer)の最初の引数として受け取ることができます。
`init`関数はモジュールごとに一度だけ呼び出されるため、OTWは一度だけインスタンス化されることが
保証されています。

## OTWの強制

型がOTWかどうかをチェックするために、[Sui Framework](./sui-framework)の`sui::types`モジュールは
型がOTWかどうかをチェックするために使用できる特別な関数`is_one_time_witness`を提供します。

```move file=packages/samples/sources/programmability/one-time-witness.move anchor=usage

```

<!-- ## 背景

OTWの実際の定義に到達する前に、簡単な例を考えてみましょう。ウィットネスで初期化できる
Coin型のジェネリック実装を構築したいと思います。ウィットネス`T`のインスタンスは、
新しい`TreasuryCap<T>`を作成するために使用され、その後新しい`Coin<T>`をミントするために
使用されます。

```move
module book::simple_coin {

    /// Coinの供給を制御します。
    public struct TreasuryCap<phantom T> has key, store {
        id: UID,
        total_supply: u64,
    }

    /// `T`がウィットネスであるCoin型。
    public struct Coin<phantom T> has key, store {
        id: UID,
        value: u64,
    }

    /// ウィットネスで新しいTreasuryCapを作成します。
    /// 脆弱：同じウィットネスで複数のTreasuryCap<T>を作成できます。
    public fun new<T: drop>(_: T, ctx: &mut TxContext): TreasuryCap<T> {
        TreasuryCap { id: object::new(ctx), total_supply: 0 }
    }

    /// ミンティングを承認するために通常のウィットネスを使用します。
    public fun mint<T>(
        treasury: &mut TreasuryCap<T>,
        value: u64,
        ctx: &mut TxContext
    ) {
        treasury.total_supply = treasury.total_supply + value;
        Coin { id: object::new(ctx), value }
    }
}
```

不正な開発者は、同じウィットネスで複数の`TreasuryCap`を作成し、予想よりも多くの`Coin`を
ミントできるでしょう。以下は、そのような悪意のあるモジュールの例です：

```move
module book::simple_coin_cheater {
    /// Coinウィットネス。
    public struct Move has drop {}

    /// MoveウィットネスでTreasuryCapを初期化します。
    /// ...そして2回実行します！>_<
    fun init(ctx: &mut TxContext) {
        let treasury_cap = book::simple_coin::new(Move {}, ctx);
        let secret_treasury = book::simple_coin::new(Move {}, ctx);

        transfer::public_transfer(treasury_cap, ctx.sender())
        transfer::public_transfer(secret_treasury, ctx.sender())
    }
}

```

上記の例では、同じウィットネスで複数の`TreasuryCap`を発行することに対する保護がなく、
実際のアプリケーションでは、これは信頼の問題を生み出します。この実装に基づいて
Coinをサポートするという人間の決定だった場合、以下を確認する必要があります：

- 与えられた`T`に対して`TreasuryCap`は1つだけ存在する。
- モジュールはアップグレードしてより多くの`TreasuryCap`を発行できない。
- モジュールコードには、より多くの`TreasuryCap`を発行するバックドアが含まれていない。

しかし、Moveコード内でこれらの条件のいずれかをチェックすることは不可能です。
そして、信頼の必要性を防ぐために、SuiはOTWパターンを導入します。

## Coin問題の解決

複数の`TreasuryCap`のケースを解決するために、OTWパターンを使用できます。
`COIN_OTW`型をOTWとして定義することで、`COIN_OTW`が一度だけ使用されることを
保証できます。`COIN_OTW`はその後、新しい`TreasuryCap`を作成し、新しい`Coin`を
ミントするために使用されます。

```move

With

```move
module book::coin_otw {

    /// `book::coin_otw`モジュールのOTW。
    struct COIN_OTW has drop {}

    /// 最初の引数として`COIN_OTW`のインスタンスを受け取ります。
    fun init(otw: COIN_OTW, ctx: &mut TxContext) {
        let treasury_cap = book::simple_coin::new(COIN_OTW {}, ctx);
        transfer::public_transfer(treasury_cap, ctx.sender())
    }
}
```


 -->

<!-- ## ケーススタディ：Coin

TODO: TreasuryCapとCoinの背後にあるストーリーを追加

-->

## まとめ

OTWパターンは、型が一度だけ使用されることを保証する優れた方法です。ほとんどの開発者は
OTWを定義し受け取る方法を理解する必要がありますが、OTWチェックと強制は主に
ライブラリとフレームワークで必要です。例えば、`sui::coin`モジュールは
`coin::create_currency`メソッドでOTWを要求するため、`coin::TreasuryCap`が一度だけ
作成されることを強制します。

OTWは、次のセクションで説明する[Publisher](./publisher)オブジェクトの基盤を築く
強力なツールです。

<!--

## 質問
- 複数の`TreasuryCap`を防ぐために他にどのような方法が使用できるか？
- OTWを使用する他の方法はあるか？

 -->
