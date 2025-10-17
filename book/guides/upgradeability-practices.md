# アップグレード可能性の実践

アップグレード可能性のベストプラクティスについて話すには、まずパッケージで何がアップグレードできるかを理解する必要があります。アップグレード可能性の基本前提は、アップグレードが前のバージョンとのパブリック互換性を破らないことです。依存パッケージで使用できるモジュールの部分は、静的シグネチャを変更してはいけません。これはモジュール（モジュールはパッケージから削除できません）、パブリック構造体（関数シグネチャで使用できます）、パブリック関数（他のパッケージから呼び出せます）に適用されます。

```move
// モジュールはパッケージから削除できません
module book::upgradable;

// 依存関係は変更できます（パブリックシグネチャで使用されていない場合）
use std::string::String;
use sui::event; // 削除可能

// パブリック構造体は削除できず、変更できません
public struct Book has key {
    id: UID,
    title: String,
}

// パブリック構造体は削除できず、変更できません
public struct BookCreated has copy, drop {
    /* ... */
}

// パブリック関数は削除できず、シグネチャは決して変更できません
// しかし、実装は変更できます
public fun create_book(ctx: &mut TxContext): Book {
    create_book_internal(ctx)

    // 削除および変更可能
    event::emit(BookCreated {
        /* ... */
    })
}

// パッケージ可視性関数は削除および変更可能
public(package) fun create_book_package(ctx: &mut TxContext): Book {
    create_book_internal(ctx)
}

// エントリ関数は、パブリックでない限り削除および変更可能
entry fun create_book_entry(ctx: &mut TxContext): Book {
    create_book_internal(ctx)
}

// プライベート関数は削除および変更可能
fun create_book_internal(ctx: &mut TxContext): Book {
    abort
}
```

<!--
## エントリとフレンド関数の使用

TODO: エントリとフレンド関数についてのセクションを追加
-->

## オブジェクトのバージョニング

<!-- この実践は、共有状態に基づく関数バージョンロック用です -->

パッケージの以前のバージョンを破棄するために、オブジェクトをバージョニングできます。オブジェクトにバージョンフィールドが含まれ、オブジェクトを使用するコードが特定のバージョンを期待し、アサートする限り、コードを新しいバージョンに強制移行できます。通常、アップグレード後、管理者関数を使用して共有状態のバージョンを更新し、新しいバージョンのコードを使用できるようにし、古いバージョンはバージョンの不一致でアボートするようにできます。

```move
module book::versioned_state;

const EVersionMismatch: u64 = 0;

const VERSION: u8 = 1;

/// 共有状態（所有することも可能）
public struct SharedState has key {
    id: UID,
    version: u8,
    /* ... */
}

public fun mutate(state: &mut SharedState) {
    assert!(state.version == VERSION, EVersionMismatch);
    // ...
}
```

## 動的フィールドによる設定のバージョニング

<!-- この実践は、オブジェクトの内容/構造のバージョニング用です -->

Suiには、同じオブジェクトシグネチャを保持しながら、オブジェクトの保存された設定を変更できる一般的なパターンがあります。これは、ベースオブジェクトをシンプルでバージョン管理された状態に保ち、実際の設定オブジェクトを動的フィールドとして追加することで実現されます。この_アンカー_パターンを使用することで、同じベースオブジェクトシグネチャを保持しながら、パッケージアップグレードで設定を変更できます。

```move
module book::versioned_config;

use sui::vec_map::VecMap;
use std::string::String;

/// ベースオブジェクト
public struct Config has key {
    id: UID,
    version: u16
}

/// 実際の設定
public struct ConfigV1 has store {
    data: Bag,
    metadata: VecMap<String, String>
}

// ...
```

<!-- ## モジュラーアーキテクチャ -->

<!-- TODO: モジュラーアーキテクチャの2つのパターンを追加：オブジェクト能力（SuiFrens）とウィットネスレジストリ（SuiNS） -->