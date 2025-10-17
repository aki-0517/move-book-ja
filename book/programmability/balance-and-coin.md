# バランス & コイン

Suiでは、コインとバランスは重要な概念です。このセクションでは、コインの作成、管理、転送について説明します。

## コインの基本概念

コインは、Sui上で価値を表現するオブジェクトです。各コインは特定の型（例：SUI、USDCなど）を持ち、所有者がいます。

## バランス

バランスは、特定の型のコインの総量を追跡するために使用されます。バランスは、コインの供給量を管理するために使用されます。

```move
module book::coin_example {
    use sui::coin::{Self, Coin};
    use sui::balance::{Self, Balance};
    use sui::tx_context::TxContext;
    
    public fun create_coin<T>(ctx: &mut TxContext): Coin<T> {
        // 新しいコインを作成
        coin::mint<T>(100, ctx)
    }
    
    public fun get_balance<T>(coin: &Coin<T>): u64 {
        coin::value(coin)
    }
}
```

## コインの転送

コインは、所有者間で転送することができます。転送は、`transfer`関数を使用して実行されます。

```move
module book::coin_transfer {
    use sui::coin::{Self, Coin};
    use sui::transfer;
    use sui::tx_context::TxContext;
    
    public fun transfer_coin<T>(coin: Coin<T>, recipient: address) {
        transfer::public_transfer(coin, recipient);
    }
}
```

## まとめ

コインとバランスは、Suiアプリケーションで価値を管理するための基本的なツールです。適切に使用することで、安全で効率的なトークンシステムを構築できます。
