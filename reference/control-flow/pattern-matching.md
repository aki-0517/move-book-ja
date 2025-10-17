---
title: 'パターンマッチング | リファレンス'
description: ''
---

# パターンマッチング

`match`式は、値を一連のパターンと比較し、最初にマッチするパターンに基づいてコードを実行することを可能にする強力な制御構造です。パターンは、単純なリテラルから複雑なネストした構造体や列挙型定義まで何でも可能です。`bool`型のテスト式に基づいて制御フローを変更する`if`式とは対照的に、`match`式は任意の型の値に対して動作し、多くの分岐から1つを選択します。

`match`式は、Move値だけでなく、可変または不変の参照もマッチでき、それに応じてサブパターンをバインドします。

For example:

```move
fun run(x: u64): u64 {
    match (x) {
        1 => 2,
        2 => 3,
        x => x,
    }
}

run(1); // returns 2
run(2); // returns 3
run(3); // returns 3
run(0); // returns 0
```

## `match`構文

`match`は式と、カンマで区切られた空でない一連の_マッチアーム_を取ります。

各マッチアームは、パターン（`p`）、オプションのガード（`g`が`bool`型の式である`if (g)`）、矢印（`=>`）、およびパターンがマッチしたときに実行するアーム式（`e`）で構成されます。例えば、

```move
match (expression) {
    pattern1 if (guard_expression) => expression1,
    pattern2 => expression2,
    pattern3 => { expression3, expression4, ... },
}
```

マッチアームは上から下の順序でチェックされ、マッチする最初のパターン（存在する場合、ガード式が`true`と評価される）が実行されます。

`match`内のマッチアームの系列は網羅的でなければならないことに注意してください。つまり、マッチされる型のすべての可能な値が`match`のパターンのいずれかでカバーされる必要があります。マッチアームの系列が網羅的でない場合、コンパイラはエラーを発生させます。

## パターン構文

パターンは、値がパターンと等しい場合に値によってマッチされ、変数とワイルドカード（例：`x`、`y`、`_`、または`..`）は何にでも「等しい」とされます。

パターンは値をマッチするために使用されます。パターンは以下のようになります：

| Pattern              | Description                                                            |
| -------------------- | ---------------------------------------------------------------------- |
| Literal              | A literal value, such as `1`, `true`, `@0x1`                           |
| Constant             | A constant value, e.g., `MyConstant`                                   |
| Variable             | A variable, e.g., `x`, `y`, `z`                                        |
| Wildcard             | A wildcard, e.g., `_`                                                  |
| Constructor          | A constructor pattern, e.g., `MyStruct { x, y }`, `MyEnum::Variant(x)` |
| At-pattern           | An at-pattern, e.g., `x @ MyEnum::Variant(..)`                         |
| Or-pattern           | An or-pattern, e.g., `MyEnum::Variant(..) \| MyEnum::OtherVariant(..)` |
| Multi-arity wildcard | A multi-arity wildcard, e.g., `MyEnum::Variant(..)`                    |
| Mutable-binding      | A mutable-binding pattern, e.g., `mut x`                               |

Moveのパターンは以下の文法を持ちます：

```bnf
pattern = <literal>
        | <constant>
        | <variable>
        | _
        | C { <variable> : inner-pattern ["," <variable> : inner-pattern]* } // where C is a struct or enum variant
        | C ( inner-pattern ["," inner-pattern]* ... )                       // where C is a struct or enum variant
        | C                                                                  // where C is an enum variant
        | <variable> @ top-level-pattern
        | pattern | pattern
        | mut <variable>
inner-pattern = pattern
              | ..     // multi-arity wildcard
```

パターンの例は以下の通りです：

```move
// literal pattern
1

// constant pattern
MyConstant

// variable pattern
x

// wildcard pattern
_

// constructor pattern that matches `MyEnum::Variant` with the fields `1` and `true`
MyEnum::Variant(1, true)

// constructor pattern that matches `MyEnum::Variant` with the fields `1` and binds the second field's value to `x`
MyEnum::Variant(1, x)

// multi-arity wildcard pattern that matches multiple fields within the `MyEnum::Variant` variant
MyEnum::Variant(..)

// constructor pattern that matches the `x` field of `MyStruct` and binds the `y` field to `other_variable`
MyStruct { x, y: other_variable }

// at-pattern that matches `MyEnum::Variant` and binds the entire value to `x`
x @ MyEnum::Variant(..)

// or-pattern that matches either `MyEnum::Variant` or `MyEnum::OtherVariant`
MyEnum::Variant(..) | MyEnum::OtherVariant(..)

// same as the above or-pattern, but with explicit wildcards
MyEnum::Variant(_, _) | MyEnum::OtherVariant(_, _)

// or-pattern that matches either `MyEnum::Variant` or `MyEnum::OtherVariant` and binds the u64 field to `x`
MyEnum::Variant(x, _) | MyEnum::OtherVariant(_, x)

// constructor pattern that matches `OtherEnum::V` and if the inner `MyEnum` is `MyEnum::Variant`
OtherEnum::V(MyEnum::Variant(..))
```

### パターンと変数

変数を含むパターンは、それらをマッチ対象またはマッチ対象のサブコンポーネントにバインドします。これらの変数は、任意のマッチガード式で、またはマッチアームの右側で使用できます。例えば：

```move
public struct Wrapper(u64)

fun add_under_wrapper_unless_equal(wrapper: Wrapper, x: u64): Wrapper {
    match (wrapper) {
        Wrapper(y) if (y == x) => Wrapper(y),
        Wrapper(y) => Wrapper(y + x),
    }
}
add_under_wrapper_unless_equal(Wrapper(1), 2); // returns Wrapper(3)
add_under_wrapper_unless_equal(Wrapper(2), 3); // returns Wrapper(5)
add_under_wrapper_unless_equal(Wrapper(3), 3); // returns Wrapper(3)
```

### パターンの組み合わせ

パターンはネストできますが、or演算子（`|`）を使用してパターンを組み合わせることもできます。例えば、`p1 | p2`は、パターン`p1`または`p2`のいずれかが対象にマッチした場合に成功します。このパターンはどこでも発生する可能性があります -- トップレベルのパターンとして、または別のパターン内のサブパターンとして。

```move
public enum MyEnum has drop {
    Variant(u64, bool),
    OtherVariant(bool, u64),
}

fun test_or_pattern(x: u64): u64 {
    match (x) {
        MyEnum::Variant(1 | 2 | 3, true) | MyEnum::OtherVariant(true, 1 | 2 | 3) => 1,
        MyEnum::Variant(8, true) | MyEnum::OtherVariant(_, 6 | 7) => 2,
        _ => 3,
    }
}

test_or_pattern(MyEnum::Variant(3, true)); // returns 1
test_or_pattern(MyEnum::OtherVariant(true, 2)); // returns 1
test_or_pattern(MyEnum::Variant(8, true)); // returns 2
test_or_pattern(MyEnum::OtherVariant(false, 7)); // returns 2
test_or_pattern(MyEnum::OtherVariant(false, 80)); // returns 3
```

### 一部のパターンの制限

`mut`と`..`パターンには、いつ、どこで、どのように使用できるかについて特定の条件が課されています。詳細は[特定のパターンの制限](#limitations-on-specific-patterns)を参照してください。高レベルでは、`mut`修飾子は変数パターンでのみ使用でき、`..`パターンはコンストラクタパターン内で一度だけ使用でき、トップレベルのパターンとしては使用できません。

以下は、`..`パターンの_無効な_使用例です。トップレベルのパターンとして使用されているためです：

```move
match (x) {
    .. => 1,
    // ERROR: `..` pattern can only be used within a constructor pattern
}

match (x) {
    MyStruct(.., ..) => 1,
    // ERROR:    ^^  `..` pattern can only be used once within a constructor pattern
}
```

### パターンの型付け

パターンは式ではありませんが、それでも型付けされています。これは、パターンの型がマッチする値の型と一致する必要があることを意味します。例えば、パターン`1`は整数型を持ち、パターン`MyEnum::Variant(1, true)`は型`MyEnum`を持ち、パターン`MyStruct { x, y }`は型`MyStruct`を持ち、`OtherStruct<bool> { x: true, y: 1}`は型`OtherStruct<bool>`を持ちます。マッチ内のパターンの型と異なる式でマッチしようとすると、型エラーが発生します。例えば：

```move
match (1) {
    // The `true` literal pattern is of type `bool` so this is a type error.
    true => 1,
    // TYPE ERROR: expected type u64, found bool
    _ => 2,
}
```

同様に、`MyEnum`と`MyStruct`は異なる型であるため、以下も型エラーになります：

```move
match (MyStruct { x: 0, y: 0 }) {
    MyEnum::Variant(..) => 1,
    // TYPE ERROR: expected type MyEnum, found MyStruct
}
```

## マッチング

パターンマッチングの詳細と値がパターンに「マッチ」するという意味について詳しく説明する前に、概念の直感を提供するためにいくつかの例を調べてみましょう。

```move
fun test_lit(x: u64): u8 {
    match (x) {
        1 => 2,
        2 => 3,
        _ => 4,
    }
}
test_lit(1); // returns 2
test_lit(2); // returns 3
test_lit(3); // returns 4
test_lit(10); // returns 4

fun test_var(x: u64): u64 {
    match (x) {
        y => y,
    }
}
test_var(1); // returns 1
test_var(2); // returns 2
test_var(3); // returns 3
...

const MyConstant: u64 = 10;
fun test_constant(x: u64): u64 {
    match (x) {
        MyConstant => 1,
        _ => 2,
    }
}
test_constant(MyConstant); // returns 1
test_constant(10); // returns 1
test_constant(20); // returns 2

fun test_or_pattern(x: u64): u64 {
    match (x) {
        1 | 2 | 3 => 1,
        4 | 5 | 6 => 2,
        _ => 3,
    }
}
test_or_pattern(3); // returns 1
test_or_pattern(5); // returns 2
test_or_pattern(70); // returns 3

fun test_or_at_pattern(x: u64): u64 {
    match (x) {
        x @ (1 | 2 | 3) => x + 1,
        y @ (4 | 5 | 6) => y + 2,
        z => z + 3,
    }
}
test_or_pattern(2); // returns 3
test_or_pattern(5); // returns 7
test_or_pattern(70); // returns 73
```

これらの例から最も重要なことは、値がパターンと等しい場合にパターンが値にマッチし、ワイルドカード/変数パターンが何にでもマッチするということです。これはリテラル、変数、定数に当てはまります。例えば、`test_lit`関数では、値`1`はパターン`1`にマッチし、値`2`はパターン`2`にマッチし、値`3`はワイルドカード`_`にマッチします。同様に、`test_var`関数では、値`1`と値`2`の両方がパターン`y`にマッチします。

変数`x`は任意の値にマッチ（または「等しい」）し、ワイルドカード`_`は任意の値にマッチします（ただし1つの値のみ）。Or-patternは論理ORのようなもので、値がor-pattern内の任意のパターンにマッチした場合にパターンにマッチするため、`p1 | p2 | p3`は「p1、またはp2、またはp3にマッチ」と読むべきです。

### コンストラクタのマッチング

パターンマッチングにはコンストラクタパターンの概念が含まれます。これらのパターンにより、構造体と列挙型の両方の深い部分を検査し、アクセスでき、パターンマッチングの最も強力な部分の一つです。コンストラクタパターンは、変数バインディングと組み合わせることで、値の構造に基づいて値をマッチし、マッチアームの右側で使用するために必要な値の部分を取り出すことができます。

以下を考えてみましょう：

```move
fun f(x: MyEnum): u64 {
    match (x) {
        MyEnum::Variant(1, true) => 1,
        MyEnum::OtherVariant(_, 3) => 2,
        MyEnum::Variant(..) => 3,
        MyEnum::OtherVariant(..) => 4,
    }
}
f(MyEnum::Variant(1, true)); // returns 1
f(MyEnum::Variant(2, true)); // returns 3
f(MyEnum::OtherVariant(false, 3)); // returns 2
f(MyEnum::OtherVariant(true, 3)); // returns 2
f(MyEnum::OtherVariant(true, 2)); // returns 4
```

これは「`x`がフィールド`1`と`true`を持つ`MyEnum::Variant`の場合、`1`を返す。最初のフィールドに任意の値を持ち、2番目に`3`を持つ`MyEnum::OtherVariant`の場合、`2`を返す。任意のフィールドを持つ`MyEnum::Variant`の場合、`3`を返す。最後に、任意のフィールドを持つ`MyEnum::OtherVariant`の場合、`4`を返す」と言っています。

パターンをネストすることもできます。したがって、前の`MyEnum::Variant`で単に1をマッチする代わりに、1、2、または10のいずれかにマッチしたい場合は、or-patternで行うことができます：

```move
fun f(x: MyEnum): u64 {
    match (x) {
        MyEnum::Variant(1 | 2 | 10, true) => 1,
        MyEnum::OtherVariant(_, 3) => 2,
        MyEnum::Variant(..) => 3,
        MyEnum::OtherVariant(..) => 4,
    }
}
f(MyEnum::Variant(1, true)); // returns 1
f(MyEnum::Variant(2, true)); // returns 1
f(MyEnum::Variant(10, true)); // returns 1
f(MyEnum::Variant(10, false)); // returns 3
```

### 能力制約

さらに、マッチバインディングは、Moveの他の側面と同じ能力制限の対象となります。特に、ワイルドカードを使用して`drop`のない値（非参照）をマッチしようとすると、ワイルドカードが値をドロップすることを期待するため、コンパイラはエラーを発生させます。同様に、バインダーを使用して非`drop`値をバインドする場合、マッチアームの右側で使用する必要があります。さらに、その値を完全に破棄する場合、[非`drop`構造体のアンパック](./../structs#destroying-structs-via-pattern-matching)のセマンティクスに一致するようにアンパックされています。`drop`機能の詳細については、[`drop`の能力セクション](./../abilities#drop)を参照してください。

```move
public struct NonDrop(u64)

fun drop_nondrop(x: NonDrop): u64 {
    match (x) {
        NonDrop(1) => 1,
        _ => 2
        // ERROR: cannot wildcard match on a non-droppable value
    }
}

fun destructure_nondrop(x: NonDrop): u64 {
    match (x) {
        NonDrop(1) => 1,
        NonDrop(_) => 2
        // OK!
    }
}

fun use_nondrop(x: NonDrop): NonDrop {
    match (x) {
        NonDrop(1) => NonDrop(8),
        x => x
    }
}
```

## 網羅性

Moveの`match`式は_網羅的_でなければなりません：マッチされる型のすべての可能な値が、マッチのアームのいずれかのパターンでカバーされる必要があります。マッチアームの系列が網羅的でない場合、コンパイラはエラーを発生させます。ガード式を持つアームは、実行時にマッチに失敗する可能性があるため、マッチの網羅性に寄与しないことに注意してください。

例として、`u8`のマッチは、ワイルドカードまたは変数パターンが存在しない限り、0から255までの_すべての_数値にマッチする場合にのみ網羅的です。同様に、`bool`のマッチは、ワイルドカードまたは変数パターンが存在しない限り、`true`と`false`の両方にマッチする必要があります。

構造体の場合、その型には1つのコンストラクタタイプしかないため、1つのコンストラクタをマッチするだけで済みますが、構造体内のフィールドも網羅的にマッチする必要があります。逆に、列挙型は複数のバリアントを定義でき、マッチが網羅的と見なされるためには、各バリアント（サブフィールドを含む）をマッチする必要があります。

アンダースコアと変数は何にでもマッチするワイルドカードであるため、その位置でマッチしている型のすべての値にマッチするとカウントされます。さらに、多項ワイルドカードパターン`..`を使用して、構造体または列挙型バリアント内の複数の値にマッチできます。

_非網羅的_マッチの例を見るために、以下を考えてみましょう：

```move
public enum MyEnum {
    Variant(u64, bool),
    OtherVariant(bool, u64),
}

public struct Pair<T>(T, T)

fun f(x: MyEnum): u8 {
    match (x) {
        MyEnum::Variant(1, true) => 1,
        MyEnum::Variant(_, _) => 1,
        MyEnum::OtherVariant(_, 3) => 2,
        // ERROR: not exhaustive as the value `MyEnum::OtherVariant(_, 4)` is not matched.
    }
}

fun match_pair_bool(x: Pair<bool>): u8 {
    match (x) {
        Pair(true, true) => 1,
        Pair(true, false) => 1,
        Pair(false, false) => 1,
        // ERROR: not exhaustive as the value `Pair(false, true)` is not matched.
    }
}
```

これらの例は、マッチアームの最後にワイルドカードパターンを追加するか、残りの値に完全にマッチすることで網羅的にすることができます：

```move
fun f(x: MyEnum): u8 {
    match (x) {
        MyEnum::Variant(1, true) => 1,
        MyEnum::Variant(_, _) => 1,
        MyEnum::OtherVariant(_, 3) => 2,
        // Now exhaustive since this will match all values of MyEnum::OtherVariant
        MyEnum::OtherVariant(..) => 2,

    }
}

fun match_pair_bool(x: Pair<bool>): u8 {
    match (x) {
        Pair(true, true) => 1,
        Pair(true, false) => 1,
        Pair(false, false) => 1,
        // Now exhaustive since this will match all values of Pair<bool>
        Pair(false, true) => 1,
    }
}
```

## ガード

前述のように、パターンの後に`if`句を追加することで、マッチアームにガードを追加できます。このガードは、パターンがマッチされた_後_に実行されますが、矢印の右側の式が評価される_前_に実行されます。ガード式が`true`と評価された場合、矢印の右側の式が評価され、`false`と評価された場合、マッチに失敗したと見なされ、`match`式の次のマッチアームがチェックされます。

```move
fun match_with_guard(x: u64): u64 {
    match (x) {
        1 if (false) => 1,
        1 => 2,
        _ => 3,
    }
}

match_with_guard(1); // returns 2
match_with_guard(0); // returns 3
```

ガード式は、評価中にパターンでバインドされた変数を参照できます。ただし、マッチされるパターンに関係なく、_変数はガード内で不変参照としてのみ利用可能_であることに注意してください -- 変数に可変性指定子がある場合や、パターンが値でマッチされている場合でも。

```move
fun incr(x: &mut u64) {
    *x = *x + 1;
}

fun match_with_guard_incr(x: u64): u64 {
    match (x) {
        x if ({ incr(&mut x); x == 1 }) => 1,
        // ERROR:    ^^^ invalid borrow of immutable value
        _ => 2,
    }
}

fun match_with_guard_incr2(x: &mut u64): u64 {
    match (x) {
        x if ({ incr(&mut x); x == 1 }) => 1,
        // ERROR:    ^^^ invalid borrow of immutable value
        _ => 2,
    }
}
```

さらに、ガード式を持つマッチアームは、コンパイラがガード式を静的に評価する方法がないため、網羅性の目的では考慮されないことに注意することが重要です。

## 特定のパターンの制限

パターンで`..`と`mut`パターン修飾子をいつ使用できるかについて、いくつかの制限があります。

### 可変性の使用

`mut`修飾子は変数パターンに配置して、_変数_がマッチアームの右側の式で変更されることを指定できます。`mut`修飾子は変数が変更されることを示すだけで、基になるデータではないため、すべてのタイプのマッチ（値、不変参照、可変参照）で使用できることに注意してください。

`mut`修飾子は変数にのみ適用でき、他のタイプのパターンには適用できないことに注意してください。

```move
public struct MyStruct(u64)

fun top_level_mut(x: MyStruct): u64 {
    match (x) {
        mut MyStruct(y) => 1,
        // ERROR: cannot use mut on a non-variable pattern
    }
}

fun mut_on_immut(x: &MyStruct): u64 {
    match (x) {
        MyStruct(mut y) => {
            y = &(*y + 1);
            *y
        }
    }
}

fun mut_on_value(x: MyStruct): u64 {
    match (x) {
        MyStruct(mut y) => {
            *y = *y + 1;
            *y
        },
    }
}

fun mut_on_mut(x: &mut MyStruct): u64 {
    match (x) {
        MyStruct(mut y) => {
            *y = *y + 1;
            *y
        },
    }
}

let mut x = MyStruct(1);

mut_on_mut(&mut x); // returns 2
x.0; // returns 2

mut_on_immut(&x); // returns 3
x.0; // returns 2

mut_on_value(x); // returns 3
```

### `..`の使用

`..`パターンは、任意の数のフィールドにマッチするワイルドカードとして、コンストラクタパターン内でのみ使用できます -- コンパイラは`..`を、コンストラクタパターンの欠落しているフィールド（存在する場合）に`_`を挿入するように展開します。したがって、`MyStruct(_, _, _)`は`MyStruct(..)`と同じで、`MyStruct(1, _, _)`は`MyStruct(1, ..)`と同じです。このため、`..`パターンをいつ、どこで使用できるかについて、いくつかの制限があります：

- コンストラクタパターン内で**一度だけ**使用できます；
- 位置引数では、コンストラクタ内のパターンの開始、中間、または終了で使用できます；
- 名前付き引数では、コンストラクタ内のパターンの終了でのみ使用できます；

```move
public struct MyStruct(u64, u64, u64, u64) has drop;

public struct MyStruct2 {
    x: u64,
    y: u64,
    z: u64,
    w: u64,
}

fun wild_match(x: MyStruct): u64 {
    match (x) {
        MyStruct(.., 1) => 1,
        // OK! The `..` pattern can be used at the beginning of the constructor pattern
        MyStruct(1, ..) => 2,
        // OK! The `..` pattern can be used at the end of the constructor pattern
        MyStruct(1, .., 1) => 3,
        // OK! The `..` pattern can be used at the middle of the constructor pattern
        MyStruct(1, .., 1, 1) => 4,
        MyStruct(..) => 5,
    }
}

fun wild_match2(x: MyStruct2): u64 {
    match (x) {
        MyStruct2 { x: 1, .. } => 1,
        MyStruct2 { x: 1, w: 2 .. } => 2,
        MyStruct2 { .. } => 3,
    }
}
```
