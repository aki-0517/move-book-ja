# コード品質チェックリスト

Move言語とそのエコシステムの急速な進化により、多くの古い実践が時代遅れになっています。
このガイドは、開発者がコードをレビューし、Move開発の現在のベストプラクティスに
合致していることを確認するためのチェックリストとして機能します。注意深く読み、
可能な限り多くの推奨事項をコードに適用してください。

## コード組織

このガイドで言及されている問題の一部は、
[Move Formatter](https://www.npmjs.com/package/@mysten/prettier-plugin-move)をCLIツールとして、
または[CIチェックとして](https://github.com/marketplace/actions/move-formatter)、
または[VSCode（Cursor）のプラグインとして](https://marketplace.visualstudio.com/items?itemName=mysten.prettier-move)
使用することで修正できます。

## パッケージマニフェスト

### 正しいエディションを使用する

このガイドのすべての機能にはMove 2024エディションが必要であり、
パッケージマニフェストで指定する必要があります。

```toml
[package]
name = "my_package"
edition = "2024.beta" # または（単に）"2024"
```

### 暗黙的なフレームワーク依存関係

Sui 1.45から、`Move.toml`でフレームワーク依存関係を指定する必要がなくなりました：

```toml
# 古い、1.45以前
[dependencies]
Sui = { ... }

# 現代では、Sui、Bridge、MoveStdlib、SuiSystemが暗黙的にインポートされます！
[dependencies]
```

### 名前付きアドレスにプレフィックスを付ける

パッケージに汎用的な名前（例：`token`）がある場合、特にプロジェクトに複数の
パッケージが含まれている場合は、名前付きアドレスにプレフィックスを追加してください：

```toml
# 悪い！何も示しておらず、競合する可能性があります
[addresses]
math = "0x0"

# 良い！プロジェクトを明確に示し、競合する可能性が低いです
[addresses]
my_protocol_math = "0x0"
```

## インポート、モジュール、定数

### モジュールラベルの使用

```move
// 悪い：インデントが増加し、レガシースタイル
module my_package::my_module {
    public struct A {}
}

// 良い！
module my_package::my_module;

public struct A {}
```

### `use`文で単一の`Self`を使用しない

```move
// 正しい、メンバー + self インポート
use my_package::other::{Self, OtherMember};

// 悪い！`{Self}`は冗長です
use my_package::my_module::{Self};

// 良い！
use my_package::my_module;
```

### `Self`で`use`文をグループ化する

```move
// 悪い！
use my_package::my_module;
use my_package::my_module::OtherMember;

// 良い！
use my_package::my_module::{Self, OtherMember};
```

### エラー定数は`EPascalCase`

```move
// 悪い！大文字は通常の定数に使用されます
const NOT_AUTHORIZED: u64 = 0;

// 良い！エラー定数であることが明確に示されています
const ENotAuthorized: u64 = 0;
```

### 通常の定数は`ALL_CAPS`

```move
// 悪い！PascalCaseはエラー定数に関連付けられています
const MyConstant: vector<u8> = b"my const";

// 良い！定数値であることが明確に示されています
const MY_CONSTANT: vector<u8> = b"my const";
```

## 構造体

### ケーパビリティは`Cap`で終わる

```move
// 悪い！ケーパビリティの場合は`Cap`サフィックスを追加してください
public struct Admin has key, store {
    id: UID,
}

// 良い！レビュアーは型から何を期待するかがわかります
public struct AdminCap has key, store {
    id: UID,
}
```

### 名前に`Potato`を使用しない

```move
// 悪い！アビリティがなく、Hot-Potato型であることはすでにわかっています
public struct PromisePotato {}

// 良い！
public struct Promise {}
```

### イベントは過去形で命名する

```move
// 悪い！この構造体が何をするかが明確ではありません
public struct RegisterUser has copy, drop { user: address }

// 良い！明確で、イベントです
public struct UserRegistered has copy, drop { user: address }
```

### 動的フィールドキーには位置構造体 + `Key`サフィックスを使用する

```move
// それほど悪くはありませんが、標準スタイルに反します
public struct DynamicField has copy, drop, store {}

// 良い！標準スタイル、Keyサフィックス
public struct DynamicFieldKey() has copy, drop, store;
```

## 関数

### `public entry`は使用せず、`public`または`entry`のみ

```move
// 悪い！トランザクションで関数を呼び出し可能にするためにentryは必要ありません
public entry fun do_something() { /* ... */ }

// 良い！public関数はより寛容で、値を返すことができます
public fun do_something_2(): T { /* ... */ }
```

### PTB用のコンポーザブル関数を書く

```move
// 悪い！コンポーザブルではなく、テストが困難！
public fun mint_and_transfer(ctx: &mut TxContext) {
    /* ... */
    transfer::transfer(nft, ctx.sender());
}

// 良い！コンポーザブル！
public fun mint(ctx: &mut TxContext): NFT { /* ... */ }

// 良い！意図的にコンポーザブルではありません
entry fun mint_and_keep(ctx: &mut TxContext) { /* ... */ }
```

### オブジェクトを最初に（Clockを除く）

```move
// 悪い！読みにくい！
public fun call_app(
    value: u8,
    app: &mut App,
    is_smth: bool,
    cap: &AppCap,
    clock: &Clock,
    ctx: &mut TxContext,
) { /* ... */ }

// 良い！
public fun call_app(
    app: &mut App,
    cap: &AppCap,
    value: u8,
    is_smth: bool,
    clock: &Clock,
    ctx: &mut TxContext,
) { /* ... */ }
```

### ケーパビリティを2番目に

```move
// 悪い！メソッドの結合性を破ります
public fun authorize_action(cap: &AdminCap, app: &mut App) { /* ... */ }

// 良い！署名でCapを可視に保ち、`.calls()`を維持します
public fun authorize_action(app: &mut App, cap: &AdminCap) { /* ... */ }
```

### フィールド名 + `_mut`でゲッターを命名する

```move
// 悪い！不要な`get_`
public fun get_name(u: &User): String { /* ... */ }

// 良い！フィールド`name`にアクセスすることが明確です
public fun name(u: &User): String { /* ... */ }

// 良い！可変参照には`_mut`を使用
public fun details_mut(u: &mut User): &mut Details { /* ... */ }
```

## 関数本体：構造体メソッド

### 一般的なコイン操作

```move
// 悪い！レガシーコード、読みにくい！
let paid = coin::split(&mut payment, amount, ctx);
let balance = coin::into_balance(paid);

// 良い！構造体メソッドで簡単になります！
let balance = payment.split(amount, ctx).into_balance();

// さらに良い（この例では - 一時的なコインを作成する必要がありません）
let balance = payment.balance_mut().split(amount);

// これもできます！
let coin = balance.into_coin(ctx);
```

### `std::string::utf8`をインポートしない

```move
// 悪い！残念ながら、非常に一般的です！
use std::string::utf8;

let str = utf8(b"hello, world!");

// 良い！
let str = b"hello, world!".to_string();

// ASCII文字列の場合も
let ascii = b"hello, world!".to_ascii_string();
```

### UIDには`delete`がある

```move
// 悪い！
object::delete(id);

// 良い！
id.delete();
```

### `ctx`には`sender()`がある

```move
// 悪い！
tx_context::sender(ctx);

// 良い！
ctx.sender()
```

### Vectorにはリテラルがある。そして関連関数も

```move
// 悪い！
let mut my_vec = vector::empty();
vector::push_back(&mut my_vec, 10);
let first_el = vector::borrow(&my_vec);
assert!(vector::length(&my_vec) == 1);

// 良い！
let mut my_vec = vector[10];
let first_el = my_vec[0];
assert!(my_vec.length() == 1);
```

### コレクションはインデックス構文をサポートする

```move
let x: VecMap<u8, String> = /* ... */;

// 悪い！
x.get(&10);
x.get_mut(&10);

// 良い！
&x[&10];
&mut x[&10];
```

## Option -> マクロ

### 破棄して関数を呼び出す

```move
// 悪い！
if (opt.is_some()) {
    let inner = opt.destroy_some();
    call_function(inner);
};

// 良い！そのためのマクロがあります！
opt.do!(|value| call_function(value));
```

### デフォルトでSomeを破棄する

```move
let opt = option::none();

// 悪い！
let value = if (opt.is_some()) {
    opt.destroy_some()
} else {
    abort EError
};

// 良い！マクロがあります！
let value = opt.destroy_or!(default_value);

// `none`でアボートすることもできます
let value = opt.destroy_or!(abort ECannotBeEmpty);
```

## ループ -> マクロ

### 操作をN回実行する

```move
// 悪い！読みにくい！
let mut i = 0;
while (i < 32) {
    do_action();
    i = i + 1;
};

// 良い！任意のuintにこのマクロがあります！
32u8.do!(|_| do_action());
```

### 反復から新しいベクターを作成する

```move
// 読みにくい！
let mut i = 0;
let mut elements = vector[];
while (i < 32) {
    elements.push_back(i);
    i = i + 1;
};

// 読みやすい！
vector::tabulate!(32, |i| i);
```

### ベクターのすべての要素に対して操作を実行する

```move
// 悪い！
let mut i = 0;
while (i < vec.length()) {
    call_function(&vec[i]);
    i = i + 1;
};

// 良い！
vec.do_ref!(|e| call_function(e));
```

### ベクターを破棄して各要素に関数を呼び出す

```move
// 悪い！
while (!vec.is_empty()) {
    call(vec.pop_back());
};

// 良い！
vec.destroy!(|e| call(e));
```

### ベクターを単一の値に畳み込む

```move
// 悪い！
let mut aggregate = 0;
let mut i = 0;

while (i < source.length()) {
    aggregate = aggregate + source[i];
    i = i + 1;
};

// 良い！
let aggregate = source.fold!(0, |acc, v| {
    acc + v
});
```

### ベクターの要素をフィルタリングする

> 注意：`source`ベクターで`T: drop`

```move
// 悪い！
let mut filtered = [];
let mut i = 0;
while (i < source.length()) {
    if (source[i] > 10) {
        filtered.push_back(source[i]);
    };
    i = i + 1;
};

// 良い！
let filtered = source.filter!(|e| e > 10);
```

## その他

### アンパックで無視される値は完全に無視できる

```move
// 悪い！非常に冗長！
let MyStruct { id, field_1: _, field_2: _, field_3: _ } = value;
id.delete();

// 良い！2024構文
let MyStruct { id, .. } = value;
id.delete();
```

## テスト

### `#[test]`と`#[expected_failure(...)]`をマージする

```move
// 悪い！
#[test]
#[expected_failure]
fun value_passes_check() {
    abort
}

// 良い！
#[test, expected_failure]
fun value_passes_check() {
    abort
}
```

### `expected_failure`テストをクリーンアップしない

```move
// 悪い！クリーンアップは必要ありません
#[test, expected_failure(abort_code = my_app::EIncorrectValue)]
fun try_take_missing_object_fail() {
    let mut test = test_scenario::begin(@0);
    my_app::call_function(test.ctx());
    test.end();
}

// 良い！テストがどこで失敗することが期待されるかがわかりやすい
#[test, expected_failure(abort_code = my_app::EIncorrectValue)]
fun try_take_missing_object_fail() {
    let mut test = test_scenario::begin(@0);
    my_app::call_function(test.ctx());

    abort // EIncorrectValueと異なります
}
```

### テストモジュールでテストに`test_`プレフィックスを付けない

```move
// 悪い！モジュールはすでに_testsと呼ばれています
module my_package::my_module_tests;

#[test]
fun test_this_feature() { /* ... */ }

// 良い！結果としてより良い関数名
#[test]
fun this_feature_works() { /* ... */ }
```

### 必要でない場所で`TestScenario`を使用しない

```move
// 悪い！必要ありません、ctxのみを使用
let mut test = test_scenario::begin(@0);
let nft = app::mint(test.ctx());
app::destroy(nft);
test.end();

// 良い！簡単なケース用のダミーコンテキストがあります
let ctx = &mut tx_context::dummy();
app::mint(ctx).destroy();
```

### テストで`assert!`にアボートコードを使用しない

```move
// 悪い！アプリケーションのエラーコードと偶然一致する可能性があります
assert!(is_success, 0);

// 良い！
assert!(is_success);
```

### 可能な限り`assert_eq!`を使用する

```move
// 悪い！古いスタイルのコード
assert!(result == b"expected_value", 0);

// 良い！失敗した場合に両方の値を表示します
use std::unit_test::assert_eq;

assert_eq!(result, expected_value);
```

### 「ブラックホール」`destroy`関数を使用する

```move
// 悪い！
nft.destroy_for_testing();
app.destroy_for_testing();

// 良い！ - クリーンアップ用の特別な関数を定義する必要がありません
use sui::test_utils::destroy;

destroy(nft);
destroy(app);
```

## コメント

### ドキュメントコメントは`///`で始まる

```move
// 悪い！ツールはJavaDocスタイルのコメントをサポートしていません
/**
 * Cool method
 * @param ...
 */
public fun do_something() { /* ... */ }

// 良い！docgenとIDEでドキュメントコメントとしてレンダリングされます
/// Cool method!
public fun do_something() { /* ... */ }
```

### 複雑なロジック？コメント`//`を残す

親切で、レビュアーがコードを理解するのを助けましょう！

```move
// 良い！
// 注意：値が10より小さい場合、アンダーフローする可能性があります。
// TODO: ここに`assert!`を追加する
let value = external_call(value, ctx);
```
