# 認証パターン

認証パターンは、Suiアプリケーションでアクセス制御を実装するための重要な概念です。このセクションでは、MoveとSuiで使用される主要な認証パターンについて説明します。

## アドレスチェック

最も基本的な認証方法は、トランザクションの送信者アドレスをチェックすることです。これは、特定のアドレスからのみ実行を許可したい場合に使用されます。

```move
module book::auth_example {
    use sui::tx_context::TxContext;
    
    public fun admin_only_action(ctx: &TxContext) {
        let sender = tx_context::sender(ctx);
        assert!(sender == @0x123, 0); // 特定のアドレスのみ許可
        // 管理者のみが実行できる処理
    }
}
```

## キャパビリティパターン

キャパビリティパターンは、より柔軟で安全な認証方法を提供します。オブジェクトとして表現されるキャパビリティを使用して、特定の操作を実行する権限を証明します。

```move
module book::capability_example {
    use sui::object::{Self, UID};
    use sui::transfer;
    use sui::tx_context::TxContext;
    
    public struct AdminCap has key, store {
        id: UID,
    }
    
    public fun admin_only_action(_: &AdminCap) {
        // 管理者キャパビリティを持つユーザーのみが実行可能
    }
}
```

## パブリッシャーオブジェクト

パブリッシャーオブジェクトは、パッケージの作成者が特定の型に対する権限を証明するために使用されます。

```move
module book::publisher_example {
    use sui::package::Publisher;
    use sui::tx_context::TxContext;
    
    public fun publisher_only_action(publisher: &Publisher) {
        // パブリッシャーのみが実行可能
    }
}
```

## まとめ

認証パターンは、Suiアプリケーションのセキュリティを確保するために不可欠です。適切な認証メカニズムを選択することで、アプリケーションのセキュリティと使いやすさのバランスを取ることができます。
