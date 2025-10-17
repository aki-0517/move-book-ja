---
title: '列挙型 | リファレンス'
description: ''
---

# 列挙型

_enum_（列挙型）は、1つ以上の_バリアント_を含むユーザー定義のデータ構造です。各バリアントはオプションで型付きフィールドを含むことができます。これらのフィールドの数と型は、列挙型の各バリアントで異なる可能性があります。列挙型のフィールドは、他の構造体や列挙型を含む、参照でない、タプルでない任意の型を保存できます。

簡単な例として、Moveでの以下の列挙型定義を考えてみてください：

```move
public enum Action {
    Stop,
    Pause { duration: u32 },
    MoveTo { x: u64, y: u64 },
    Jump(u64),
}
```

これは、ゲームで実行できる異なるアクションを表す列挙型`Action`を宣言しています。`Stop`、指定された期間`Pause`、特定の場所へ`MoveTo`、または特定の高さへ`Jump`することができます。

構造体と同様に、列挙型にはどのような操作を実行できるかを制御する[アビリティ](./abilities)を持たせることができます。ただし、列挙型はトップレベルオブジェクトになることができないため、`key`アビリティを持つことはできないことに注意することが重要です。

## 列挙型の定義

列挙型はモジュール内で定義される必要があり、列挙型は少なくとも1つのバリアントを含む必要があり、列挙型の各バリアントはフィールドなし、位置フィールド、または名前付きフィールドを持つことができます。以下にそれぞれの例を示します：

```move
module a::m;

public enum Foo has drop {
    VariantWithNoFields,
    //                 ^ 注：バリアント宣言の後に末尾コンマを付けても問題ありません
}
public enum Bar has copy, drop {
    VariantWithPositionalFields(u64, bool),
}
public enum Baz has drop {
    VariantWithNamedFields { x: u64, y: bool, z: Bar },
}
```

列挙型はどのバリアントでも再帰的になることはできません。そのため、以下の列挙型定義は少なくとも1つのバリアントで再帰的になるため許可されません。

不正な例：

```move
module a::m;

public enum Foo {
    Recursive(Foo),
    //        ^ エラー: 再帰的列挙型バリアント
}
public enum List {
    Nil,
    Cons { head: u64, tail: List },
    //                      ^ エラー: 再帰的列挙型バリアント
}
public enum BTree<T> {
    Leaf(T),
    Node { left: BTree<T>, right: BTree<T> },
    //           ^ エラー: 再帰的列挙型バリアント
}

// 相互再帰的列挙型も許可されません
public enum MutuallyRecursiveA {
    Base,
    Other(MutuallyRecursiveB),
    //    ^^^^^^^^^^^^^^^^^^ エラー: 再帰的列挙型バリアント
}

public enum MutuallyRecursiveB {
    Base,
    Other(MutuallyRecursiveA),
    //    ^^^^^^^^^^^^^^^^^^ エラー: 再帰的列挙型バリアント
}
```

## 可視性

すべての列挙型は`public`として宣言されます。これは、列挙型の型を他のモジュールから参照できることを意味します。ただし、列挙型のバリアント、各バリアント内のフィールド、および列挙型のバリアントを作成または破棄する能力は、列挙型を定義するモジュールの内部に留まります。

### アビリティ

構造体と同様に、デフォルトでは列挙型宣言は線形で一時的です。列挙型の値を非線形または非一時的な方法で使用する（つまり、コピー、ドロップ、または[オブジェクト](./abilities/object)に保存する）には、`has <ability>`で注釈を付けることで追加の[アビリティ](./abilities)を付与する必要があります：

```move
module a::m;

public enum Foo has copy, drop {
    VariantWithNoFields,
}
```

アビリティ宣言は列挙型のバリアントの前または後に配置できますが、どちらか一方のみを使用でき、両方は使用できません。バリアントの後に宣言する場合、アビリティ宣言はセミコロンで終了する必要があります：

```move
module a::m;

public enum PreNamedAbilities has copy, drop { Variant }
public enum PostNamedAbilities { Variant } has copy, drop;
public enum PostNamedAbilitiesInvalid { Variant } has copy, drop
//                                                              ^ エラー! セミコロンが不足
public enum NamedInvalidAbilities has copy { Variant } has drop;
//                                                     ^ エラー! 重複するアビリティ宣言
```

詳細については、[アビリティの注釈](./abilities#annotating-structs-and-enums)セクションをご覧ください。

## 命名

列挙型と列挙型内のバリアントは大文字の`A`から`Z`で始まる必要があります。最初の文字の後、列挙型名にはアンダースコア`_`、小文字`a`から`z`、大文字`A`から`Z`、または数字`0`から`9`を含めることができます。

```move
public enum Foo { Variant }
public enum BAR { Variant }
public enum B_a_z_4_2 { V_a_riant_0 }
```

`A`から`Z`で始まるという命名制限は、将来の言語機能の余地を与えるために設けられています。

## 列挙型の使用

### 列挙型バリアントの作成

列挙型の値は、列挙型のバリアントを指定し、その後にバリアント内の各フィールドの値を続けることで作成（または「パック」）できます。バリアント名は常に列挙型の名前で修飾する必要があります。

構造体と同様に、名前付きフィールドを持つバリアントの場合、フィールドの順序は重要ではありませんが、フィールド名を提供する必要があります。位置フィールドを持つバリアントの場合、フィールドの順序が重要で、フィールドの順序はバリアント宣言の順序と一致する必要があります。また、`{}`の代わりに`()`を使用して作成する必要があります。バリアントにフィールドがない場合、バリアント名だけで十分で、`()`や`{}`を使用する必要はありません。

```move
module a::m;

public enum Action has drop {
    Stop,
    Pause { duration: u32 },
    MoveTo { x: u64, y: u64 },
    Jump(u64),
}
public enum Other has drop {
    Stop(u64),
}

fun example() {
    // 注意: `Action`の`Stop`バリアントにはフィールドがないため、括弧や中括弧は不要です。
    let stop = Action::Stop;
    let pause = Action::Pause { duration: 10 };
    let move_to = Action::MoveTo { x: 10, y: 20 };
    let jump = Action::Jump(10);
    // 注意: `Other`の`Stop`バリアントには位置フィールドがあるため、それらを提供する必要があります。
    let other_stop = Other::Stop(10);
}
```

名前付きフィールドを持つバリアントの場合、構造体でおなじみの省略構文を使用してバリアントを作成することもできます：

```move
let duration = 10;

let pause = Action::Pause { duration: duration };
// 以下と同等
let pause = Action::Pause { duration };
```

### 列挙型バリアントのパターンマッチングと分解

列挙型の値は異なる形状を取ることができるため、構造体フィールドのようにバリアントのフィールドへのドットアクセスは許可されません。代わりに、バリアント内のフィールドにアクセスするには（値、不変参照、または可変参照のいずれかで）パターンマッチングを使用する必要があります。

Moveの値に対して値、不変参照、可変参照でパターンマッチングできます。値によるパターンマッチングでは、値はマッチアームにムーブされます。参照によるパターンマッチングでは、値はマッチアームに借用されます（不変または可変）。ここでは`match`を使用したパターンマッチングの簡単な説明を行いますが、Moveでの`match`を使用したパターンマッチングの詳細については[パターンマッチング](./control-flow/pattern-matching)セクションをご覧ください。

`match`文はMoveの値に対してパターンマッチングを行うために使用され、複数の_マッチアーム_で構成されます。各マッチアームは、パターン、矢印`=>`、式、そしてカンマ`,`で構成されます。パターンは構造体、列挙型バリアント、バインディング（`x`、`y`）、ワイルドカード（`_`または`..`）、定数（`ConstValue`）、またはリテラル値（`true`、`42`など）にすることができます。値は上から下に向かって各パターンと照合され、構造的に値と一致する最初のパターンにマッチします。値がマッチすると、`=>`の右側の式が実行されます。

さらに、マッチアームには、パターンがマッチした後、式が実行される_前_にチェックされるオプションの_ガード_を持つことができます。ガードは`if`キーワードの後に`=>`の前にブール値に評価される必要がある式を続けて指定します。

```move
module a::m;

public enum Action has drop {
    Stop,
    Pause { duration: u32 },
    MoveTo { x: u64, y: u64 },
    Jump(u64),
}

public struct GameState {
    // Fields containing a game state
    character_x: u64,
    character_y: u64,
    character_height: u64,
    // ...
}

fun perform_action(stat: &mut GameState, action: Action) {
    match (action) {
        // `Stop`バリアントを処理
        Action::Stop => state.stop(),
        // `Pause`バリアントを処理
        // 期間が0の場合、何もしない
        Action::Pause { duration: 0 } => (),
        Action::Pause { duration } => state.pause(duration),
        // `MoveTo`バリアントを処理
        Action::MoveTo { x, y } => state.move_to(x, y),
        // `Jump`バリアントを処理
        // ゲームがジャンプを禁止している場合は何もしない
        Action::Jump(_) if (state.jumps_not_allowed()) => (),
        // そうでなければ、指定された高さにジャンプ
        Action::Jump(height) => state.jump(height),
    }
}
```

To see how to pattern match on an enum to update values within it mutably, let's take the following
example of a simple enum that has two variants, each with a single field. We can then write two
functions, one that only increments the value of the first variant, and another that only increments
the value of the second variant:

```move
module a::m;

public enum SimpleEnum {
    Variant1(u64),
    Variant2(u64),
}

public fun incr_enum_variant1(simple_enum: &mut SimpleEnum) {
    match (simple_enum) {
        SimpleEnum::Variant1(mut value) => *value += 1,
        _ => (),
    }
}

public fun incr_enum_variant2(simple_enum: &mut SimpleEnum) {
    match (simple_enum) {
        SimpleEnum::Variant2(mut value) => *value += 1,
        _ => (),
    }
}
```

ここで、`SimpleEnum`の値がある場合、このバリアントの値を増分するためにこれらの関数を使用できます：

```move
let mut x = SimpleEnum::Variant1(10);
incr_enum_variant1(&mut x);
assert!(x == SimpleEnum::Variant1(11));
// 異なるバリアントを増分するため、増分されません
incr_enum_variant2(&mut x);
assert!(x == SimpleEnum::Variant1(11));
```

`drop`アビリティを持たないMoveの値に対してパターンマッチングを行う場合、値は各マッチアームで消費または分解される必要があります。値がマッチアームで消費または分解されない場合、コンパイラはエラーを発生させます。これは、マッチ文ですべての可能な値が処理されることを保証するためです。

例として、以下のコードを考えてみましょう：

```move
module a::m;

public enum X { Variant { x: u64 } }

public fun bad(x: X) {
    match (x) {
        _ => (),
    // ^ エラー! 型`X`の値がこのマッチアームで消費または分解されていません
    }
}
```

これを適切に処理するには、マッチのアームで`X`とそのすべてのバリアントを分解する必要があります：

```move
module a::m;

public enum X { Variant { x: u64 } }

public fun good(x: X) {
    match (x) {
        // OK! 値が分解されているためコンパイルされます
        X::Variant { x: _ } => (),
    }
}
```

### 列挙型値の上書き

列挙型が`drop`アビリティを持っている限り、Moveの他の値と同様に、列挙型の値を同じ型の新しい値で上書きできます。

```move
module a::m;

public enum X has drop {
    A(u64),
    B(u64),
}

public fun overwrite_enum(x: &mut X) {
    *x = X::A(10);
}
```

```move
let mut x = X::B(20);
overwrite_enum(&mut x);
assert!(x == X::A(10));
```
