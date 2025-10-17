# Move 2024 移行ガイド

Move 2024は、Mysten LabsによってメンテナンスされているMove言語の新しいエディションです。このガイドは
2024エディションと以前のバージョンのMove言語の違いを理解するのに役立ちます。

> このガイドは、新しいエディションの変更点の概要を提供します。より詳細で
> 網羅的な変更点のリストについては、
> [Sui Documentation](https://docs.sui.io/guides/developer/advanced/move-2024-migration)を参照してください。

## 新しいエディションの使用

新しいエディションを使用するには、`move`ファイルでエディションを指定する必要があります。エディションは
`move`ファイルで`edition`キーワードを使用して指定されます。現在、利用可能なエディションは
`2024.beta`のみです。

```ini
edition = "2024"
# または、新機能の場合：
edition = "2024.beta"
```

## 移行ツール

Move CLIには、コードを新しいエディションに更新する移行ツールがあります。移行ツールを
使用するには、次のコマンドを実行してください：

```bash
$ sui move migrate
```

移行ツールは、`let mut`構文、構造体の新しい`public`修飾子、
`friend`宣言の代わりに`public(package)`関数の可視性を使用するようにコードを更新します。

## `let mut`による可変バインディング

Move 2024では、可変変数を宣言するための`let mut`構文が導入されました。`let mut`構文は
宣言後に変更可能な可変変数を宣言するために使用されます。

> 可変変数には`let mut`宣言が必要になりました。`mut`キーワードなしで変数を再代入しようとすると、
> コンパイラはエラーを出力します。

```move
// Move 2020
let x: u64 = 10;
x = 20;

// Move 2024
let mut x: u64 = 10;
x = 20;
```

さらに、`mut`キーワードは、タプルの分割代入と関数引数で可変変数を宣言するために使用されます。

```move
// 値で受け取り、変更する
fun takes_by_value_and_mutates(mut v: Value): Value {
    v.field = 10;
    v
}

// `mut`は変数名の前に配置する必要があります
fun destruct() {
    let (x, y) = point::get_point();
    let (mut x, y) = point::get_point();
    let (mut x, mut y) = point::get_point();
}

// 構造体のアンパック
fun unpack() {
    let Point { x, mut y } = point::get_point();
    let Point { mut x, mut y } = point::get_point();
}
```

## Friendsは非推奨

Move 2024では、`friend`キーワードは非推奨になりました。代わりに、`public(package)`
可視性修飾子を使用して、同じパッケージ内の他のモジュールから関数を参照できるようにできます。

```move
// Move 2020
friend book::friend_module;
public(friend) fun protected_function() {}

// Move 2024
public(package) fun protected_function_2024() {}
```

## 構造体の可視性

Move 2024では、構造体に可視性修飾子が追加されました。現在、利用可能な可視性修飾子は
`public`のみです。

```move
// Move 2020
struct Book {}

// Move 2024
public struct Book {}
```

## メソッド構文

新しいエディションでは、最初の引数として構造体を持つ関数は、その構造体に関連付けられます。
これは、関数がドット記法を使用して呼び出せることを意味します。型と同じモジュールで定義された
メソッドは自動的にエクスポートされます。

> メソッドが定義されているモジュールと同じモジュールで型が定義されている場合、メソッドは自動的にエクスポートされます。
> 他のモジュールで定義された型のメソッドをエクスポートすることはできません。ただし、
> モジュールスコープでメソッドの[カスタムエイリアス](#method-aliases)を作成できます。

```move
public fun count(c: &Counter): u64 { /* ... */ }

fun use_counter() {
    // move 2020
    let count = counter::count(&c);

    // move 2024
    let count = c.count();
}
```

## 組み込み型のメソッド

Move 2024では、一部のネイティブ型と標準型に関連メソッドが追加されました。例えば、
`vector`型には、ベクターをUTF8文字列に変換する`to_string`メソッドがあります。

```move
fun aliases() {
    // vector to string and ascii string
    let str: String = b"Hello, World!".to_string();
    let ascii: ascii::String = b"Hello, World!".to_ascii_string();

    // address to bytes
    let bytes = @0xa11ce.to_bytes();
}
```

組み込みエイリアスの完全なリストについては、
[Standard Library](./../move-basics/standard-library#source-code)と
[Sui Framework](./../programmability/sui-framework#source-code)のソースコードを参照してください。

## 借用演算子

一部の組み込み型は借用演算子をサポートしています。借用演算子は、指定されたインデックスの要素への
参照を取得するために使用されます。借用演算子は`[]`として定義されています。

```move
fun play_vec() {
    let v = vector[1,2,3,4];
    let first = &v[0];         // vector::borrow(v, 0)を呼び出し
    let first_mut = &mut v[0]; // vector::borrow_mut(v, 0)を呼び出し
    let first_copy = v[0];     // *vector::borrow(v, 0)を呼び出し
}
```

借用演算子をサポートする型は以下の通りです：

- `vector`
- `sui::vec_map::VecMap`
- `sui::table::Table`
- `sui::bag::Bag`
- `sui::object_table::ObjectTable`
- `sui::object_bag::ObjectBag`
- `sui::linked_table::LinkedTable`

カスタム型で借用演算子を実装するには、メソッドに`#[syntax(index)]`
属性を追加する必要があります。

```move
#[syntax(index)]
public fun borrow(c: &List<T>, key: String): &T { /* ... */ }

#[syntax(index)]
public fun borrow_mut(c: &mut List<T>, key: String): &mut T { /* ... */ }
```

## メソッドエイリアス

Move 2024では、メソッドを型に関連付けることができます。エイリアスは、モジュールに対してローカルに
任意の型に対して定義できます。または、型が同じモジュールで定義されている場合は、公開できます。

```move
// my_module.move
// ローカル：型はモジュール外部のものです
use fun my_custom_function as vector.do_magic;

// sui-framework/kiosk/kiosk.move
// エクスポート：型は同じモジュールで定義されています
public use fun kiosk_owner_cap_for as KioskOwnerCap.kiosk;
```

<!-- ## Macros

Macros are introduced in Move 2024. And `assert!` is no longer a built-in function - Instead, it's a macro.

```move
// can be called as for!(0, 10, |i| call(i));
macro fun for($start: u64, $stop: u64, $body: |u64|) {
    let mut i = $start;
    let stop = $stop;
    while (i < stop) {
        $body(i);
        i = i + 1
    }
}
```
 -->
