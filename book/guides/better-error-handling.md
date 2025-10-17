# より良いエラーハンドリング

実行がアボートに遭遇するたびに、トランザクションは失敗し、アボートコードが呼び出し元に返されます。
Move VMは、トランザクションをアボートしたモジュール名とアボートコードを返します。この動作は
トランザクションの呼び出し元にとって完全に透明ではありません。特に、単一の関数が
アボートする可能性のある同じ関数への複数の呼び出しを含む場合です。この場合、呼び出し元は
どの呼び出しがトランザクションをアボートしたかを知ることができず、問題をデバッグしたり、
ユーザーに意味のあるエラーメッセージを提供したりすることが困難になります。

```move
module book::module_a;

use book::module_b;

public fun do_something() {
    let field_1 = module_b::get_field(1); // 0でアボートする可能性があります
    /* ... 多くのロジック ... */
    let field_2 = module_b::get_field(2); // 0でアボートする可能性があります
    /* ... さらにロジック ... */
    let field_3 = module_b::get_field(3); // 0でアボートする可能性があります
}
```

上記の例は、単一の関数がアボートする可能性のある複数の呼び出しを含む場合を示しています。
`do_something`関数の呼び出し元がアボートコード`0`を受け取った場合、
どの`module_b::get_field`の呼び出しがトランザクションをアボートしたかを理解することが困難になります。
この問題に対処するために、エラーハンドリングを改善するために使用できる一般的なパターンがあります。

## ルール1：すべての可能なシナリオを処理する

操作を安全に実行できるかどうかを示すブール値を返す安全な「チェック」関数を提供することは、
良い実践とされています。`module_b`が、フィールドが存在するかどうかを示すブール値を返す
`has_field`関数を提供している場合、`do_something`関数は以下のように書き直すことができます：

```move
module book::module_a;

use book::module_b;

const ENoField: u64 = 0;

public fun do_something() {
    assert!(module_b::has_field(1), ENoField);
    let field_1 = module_b::get_field(1);
    /* ... */
    assert!(module_b::has_field(2), ENoField);
    let field_2 = module_b::get_field(2);
    /* ... */
    assert!(module_b::has_field(3), ENoField);
    let field_3 = module_b::get_field(3);
}
```

`module_b::get_field`への各呼び出しの前にカスタムチェックを追加することで、`module_a`の
開発者はエラーハンドリングを制御できます。そして、これにより2番目のルールの実装が可能になります。

## ルール2：異なるコードでアボートする

2番目のトリックは、アボートコードが呼び出し元モジュールによって処理されたら、異なるシナリオに
異なるアボートコードを使用することです。このようにして、呼び出し元モジュールはユーザーに
意味のあるエラーメッセージを提供できます。`module_a`は以下のように書き直すことができます：

```move
module book::module_a;

use book::module_b;

const ENoFieldA: u64 = 0;
const ENoFieldB: u64 = 1;
const ENoFieldC: u64 = 2;

public fun do_something() {
    assert!(module_b::has_field(1), ENoFieldA);
    let field_1 = module_b::get_field(1);
    /* ... */
    assert!(module_b::has_field(2), ENoFieldB);
    let field_2 = module_b::get_field(2);
    /* ... */
    assert!(module_b::has_field(3), ENoFieldC);
    let field_3 = module_b::get_field(3);
}
```

これで、呼び出し元モジュールはユーザーに意味のあるエラーメッセージを提供できます。呼び出し元が
アボートコード`0`を受け取った場合、「フィールド1が存在しません」と翻訳できます。呼び出し元が
アボートコード`1`を受け取った場合、「フィールド2が存在しません」と翻訳できます。以下同様です。

## ルール3：`assert`の代わりに`bool`を返す

開発者は、すべての条件をアサートして実行をアボートするパブリック関数を追加したがることがよくあります。
しかし、代わりにブール値を返す関数を作成する方が良い実践です。このようにして、
呼び出し元モジュールはエラーを処理し、ユーザーに意味のあるエラーメッセージを提供できます。

```move
module book::some_app_assert;

const ENotAuthorized: u64 = 0;

public fun do_a() {
    assert_is_authorized();
    // ...
}

public fun do_b() {
    assert_is_authorized();
    // ...
}

/// これはしないでください
public fun assert_is_authorized() {
    assert!(/* some condition */ true, ENotAuthorized);
}
```

このモジュールは以下のように書き直すことができます：

```move
module book::some_app;

const ENotAuthorized: u64 = 0;

public fun do_a() {
    assert!(is_authorized(), ENotAuthorized);
    // ...
}

public fun do_b() {
    assert!(is_authorized(), ENotAuthorized);
    // ...
}

public fun is_authorized(): bool {
    /* some condition */ true
}

// 同じ条件とアボートコードが複数の場所で使用される場合の
// コード重複を避けるために、プライベート関数をまだ使用できます
fun assert_is_authorized() {
    assert!(is_authorized(), ENotAuthorized);
}
```

これらの3つのルールを活用することで、エラーハンドリングがトランザクションの呼び出し元にとって
より透明になり、他の開発者がモジュールでカスタムアボートコードを使用できるようになります。
