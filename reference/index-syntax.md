---
title: 'インデックス構文 | リファレンス'
description: ''
---

# インデックス構文

Moveは、ネイティブMoveコードのように見え、感じられる操作を定義し、これらの操作をユーザー提供の定義に下げることを可能にする構文属性を提供します。

最初の構文メソッドである`index`は、インデックス操作に使用されるべき関数に注釈を付けることで、`m[i,j]`としてマトリックス要素にアクセスするなど、データ型のカスタムインデックスアクセサーとして使用できる操作のグループを定義することを可能にします。さらに、これらの定義は型ごとに特注であり、あなたの型を使用する任意のプログラマーに対して暗黙的に利用可能です。

## 概要と要約

まず、ベクトルのベクトルを使用して値を表現する`Matrix`型を考えてみましょう。`borrow`と`borrow_mut`関数に`index`構文注釈を使用して、以下のような小さなライブラリを書くことができます：

```move
module matrix::matrix;

public struct Matrix<T> { v: vector<vector<T>> }

#[syntax(index)]
public fun borrow<T>(s: &Matrix<T>, i: u64, j: u64): &T {
    vector::borrow(vector::borrow(&s.v, i), j)
}

#[syntax(index)]
public fun borrow_mut<T>(s: &mut Matrix<T>, i: u64, j: u64): &mut T {
    vector::borrow_mut(vector::borrow_mut(&mut s.v, i), j)
}

public fun make_matrix<T>(v: vector<vector<T>>):  Matrix<T> {
    Matrix { v }
}
```

これで、この`Matrix`型を使用する誰でも、そのインデックス構文にアクセスできます：

```move
let mut m = matrix::make_matrix(vector[
    vector[1, 0, 0],
    vector[0, 1, 0],
    vector[0, 0, 1],
]);x

let mut i = 0;
while (i < 3) {
    let mut j = 0;
    while (j < 3) {
        if (i == j) {
            assert!(m[i, j] == 1, 1);
        } else {
            assert!(m[i, j] == 0, 0);
        };
        *(&mut m[i,j]) = 2;
        j = j + 1;
    };
    i = i + 1;
}
```

## 使用方法

例が示すように、データ型とそれに関連するインデックス構文メソッドを定義すると、誰でもその型の値に対してインデックス構文を書くことでそのメソッドを呼び出すことができます：

```move
let mat = matrix::make_matrix(...);
let m_0_0 = mat[0, 0];
```

コンパイル中、コンパイラは式の位置と可変使用に基づいて、これらを適切な関数呼び出しに変換します：

```move
let mut mat = matrix::make_matrix(...);

let m_0_0 = mat[0, 0];
// translates to `copy matrix::borrow(&mat, 0, 0)`

let m_0_0 = &mat[0, 0];
// translates to `matrix::borrow(&mat, 0, 0)`

let m_0_0 = &mut mat[0, 0];
// translates to `matrix::borrow_mut(&mut mat, 0, 0)`
```

インデックス式とフィールドアクセスを混在させることもできます：

```move
public struct V { v: vector<u64> }

public struct Vs { vs: vector<V> }

fun borrow_first(input: &Vs): &u64 {
    &input.vs[0].v[0]
    // translates to `vector::borrow(&vector::borrow(&input.vs, 0).v, 0)`
}
````

### インデックス関数は柔軟な引数を取る

この章の残りの部分で説明されている定義と型の制限を除いて、Moveはインデックス構文メソッドがパラメータとして取る値に制限を設けていないことに注意してください。これにより、インデックス構文を定義する際に、インデックスが範囲外の場合にデフォルト値を取るデータ構造など、複雑なプログラム動作を実装できます：

```move
#[syntax(index)]
public fun borrow_or_set<Key: copy, Value: drop>(
    input: &mut MTable<Key, Value>,
    key: Key,
    default: Value
): &mut Value {
    if (contains(input, key)) {
        borrow(input, key)
    } else {
        insert(input, key, default);
        borrow(input, key)
    }
}
```

これで、`MTable`にインデックスする際は、デフォルト値も提供する必要があります：

```move
let string_key: String = ...;
let mut table: MTable<String, u64> = m_table::make_table();
let entry: &mut u64 = &mut table[string_key, 0];
```

この種の拡張可能な機能により、型に対して正確なインデックスインターフェースを書き、カスタム動作を具体的に強制できます。

## インデックス構文関数の定義

この強力な構文形式により、すべてのユーザー定義データ型がこのように動作することができます。ただし、定義が以下のルールに従う必要があります：

1. `#[syntax(index)]`属性は、対象型と同じモジュールで定義された指定された関数に追加されます。
1. 指定された関数は`public`の可視性を持ちます。
1. 関数は参照型を対象型（最初の引数）として取り、一致する参照型を返します（対象が`mut`の場合は`mut`）。
1. 各型には単一の可変定義と単一の不変定義のみがあります。
1. 不変版と可変版は型の一致があります：
   - 対象型は一致し、可変性のみが異なります。
   - 戻り値の型は対象型の可変性と一致します。
   - 型パラメータがある場合、両バージョン間で同一の制約を持ちます。
   - 対象型を超えるすべてのパラメータは同一です。

以下の内容と追加の例では、これらのルールについてより詳しく説明します。

### 宣言

インデックス構文メソッドを宣言するには、対象型の定義と同じモジュール内の関連する関数定義の上に`#[syntax(index)]`属性を追加します。これにより、コンパイラにその関数が指定された型のインデックスアクセサーであることを知らせます。

#### 不変アクセサー

不変インデックス構文メソッドは読み取り専用アクセス用に定義されます。対象型の不変参照を取り、要素型への不変参照を返します。`std::vector`で定義された`borrow`関数がこの例です：

```move
#[syntax(index)]
public native fun borrow<Element>(v: &vector<Element>, i: u64): &Element;
```

#### 可変アクセサー

可変インデックス構文メソッドは不変版の双対で、読み取りと書き込みの両方の操作を可能にします。対象型の可変参照を取り、要素型への可変参照を返します。`std::vector`で定義された`borrow_mut`関数がこの例です：

```move
#[syntax(index)]
public native fun borrow_mut<Element>(v: &mut vector<Element>, i: u64): &mut Element;
```

#### 可視性

インデックス関数が型が使用されるどこでも利用可能であることを保証するため、すべてのインデックス構文メソッドはpublicの可視性を持たなければなりません。これにより、Moveのモジュールとパッケージ全体でインデックス機能の使いやすさが保証されます。

#### 重複なし

上記の要件に加えて、各対象基本型は不変参照用の単一のインデックス構文メソッドと可変参照用の単一のインデックス構文メソッドの定義に制限されます。例えば、多態型の特化版を定義することはできません：

```move
#[syntax(index)]
public fun borrow_matrix_u64(s: &Matrix<u64>, i: u64, j: u64): &u64 { ... }

#[syntax(index)]
public fun borrow_matrix<T>(s: &Matrix<T>, i: u64, j: u64): &T { ... }
    // ERROR! Matrix already has a definition
    // for its immutable index syntax method
```

これにより、型のインスタンス化を調べる必要なく、常にどのメソッドが呼び出されているかを知ることができます。

### 型制約

デフォルトでは、インデックス構文メソッドには以下の型制約があります：

**その対象型（最初の引数）は、マークされた関数と同じモジュールで定義された単一の型への参照でなければなりません。** これは、タプル、型パラメータ、または値に対してインデックス構文メソッドを定義できないことを意味します：

```move
#[syntax(index)]
public fun borrow_fst(x: &(u64, u64), ...): &u64 { ... }
    // ERROR because the subject type is a tuple

#[syntax(index)]
public fun borrow_tyarg<T>(x: &T, ...): &T { ... }
    // ERROR because the subject type is a type parameter

#[syntax(index)]
public fun borrow_value(x: Matrix<u64>, ...): &u64 { ... }
    // ERROR because x is not a reference
```

**対象型は戻り値の型と可変性が一致しなければなりません。** この制限により、`&vec[i]`と`&mut vec[i]`としてインデックス式を借用する際の期待される動作を明確にできます。Moveコンパイラは可変性マーカーを使用して、適切な可変性の参照を生成するためにどの借用形式を呼び出すかを決定します。その結果、対象と戻り値の可変性が異なるインデックス構文メソッドは許可されません：

```move
#[syntax(index)]
public fun borrow_imm(x: &mut Matrix<u64>, ...): &u64 { ... }
    // ERROR! incompatible mutability
    // expected a mutable reference '&mut' return type
```

### 型互換性

不変と可変のインデックス構文メソッドペアを定義する際、それらは多くの互換性制約の対象となります：

1. 同じ数の型パラメータを取り、それらの型パラメータは同じ制約を持たなければなりません。
1. 型パラメータは名前ではなく、位置によって同じように使用されなければなりません。
1. 対象型は可変性を除いて完全に一致しなければなりません。
1. 戻り値の型は可変性を除いて完全に一致しなければなりません。
1. 他のすべてのパラメータ型は完全に一致しなければなりません。

これらの制約は、インデックス構文が可変位置または不変位置にあるかに関係なく、同じように動作することを保証するためのものです。

これらのエラーのいくつかを説明するために、前の`Matrix`定義を思い出してください：

```move
#[syntax(index)]
public fun borrow<T>(s: &Matrix<T>, i: u64, j: u64): &T {
    vector::borrow(vector::borrow(&s.v, i), j)
}
```

All of the following are type-incompatible definitions of the mutable version:

```move
#[syntax(index)]
public fun borrow_mut<T: drop>(s: &mut Matrix<T>, i: u64, j: u64): &mut T { ... }
    // ERROR! `T` has `drop` here, but no in the immutable version

#[syntax(index)]
public fun borrow_mut(s: &mut Matrix<u64>, i: u64, j: u64): &mut u64 { ... }
    // ERROR! This takes a different number of type parameters

#[syntax(index)]
public fun borrow_mut<T, U>(s: &mut Matrix<U>, i: u64, j: u64): &mut U { ... }
    // ERROR! This takes a different number of type parameters

#[syntax(index)]
public fun borrow_mut<U>(s: &mut Matrix<U>, i_j: (u64, u64)): &mut U { ... }
    // ERROR! This takes a different number of arguments

#[syntax(index)]
public fun borrow_mut<U>(s: &mut Matrix<U>, i: u64, j: u32): &mut U { ... }
    // ERROR! `j` is a different type
```

Again, the goal here is to make the usage across the immutable and mutable versions consistent. This
allows index syntax methods to work without changing out the behavior or constraints based on
mutable versus immutable usage, ultimately ensuring a consistent interface to program against.
