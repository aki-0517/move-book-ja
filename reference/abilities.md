---
title: 'アビリティ | リファレンス'
description: ''
---

# アビリティ

アビリティは、特定の型の値に対してどのようなアクションが許可されるかを制御するMoveの型システム機能です。このシステムは、値の「線形」型付け動作を細かく制御し、また値がストレージでどのように使用されるか（Moveの特定のデプロイメントによって定義される、例えばブロックチェーンでのストレージの概念）を制御します。これは、特定のバイトコード命令へのアクセスを制限することで実装されており、値がそのバイトコード命令で使用されるためには、必要なアビリティを持っている必要があります（そもそも必要な場合に限り、すべての命令がアビリティによって制限されているわけではありません）。

Suiでは、`key`は[オブジェクト](./abilities/object)を示すために使用されます。オブジェクトは、各オブジェクトが一意の32バイトIDを持つストレージの基本単位です。`store`は、オブジェクト内に保存できるデータを示すために使用され、また定義モジュールの外部に転送できる型を示すためにも使用されます。

<!-- TODO future section on detailed walk through maybe. We have some examples at the end but it might be helpful to explain why we have precisely this set of abilities

If you are already somewhat familiar with abilities from writing Move programs, but are still confused as to what is going on, it might be helpful to skip to the [motivating walkthrough](#motivating-walkthrough) section to get an idea of what the system is setup in the way that it is. -->

## 4つのアビリティ

4つのアビリティは以下の通りです：

- [`copy`](#copy)
  - このアビリティを持つ型の値をコピーすることを許可します。
- [`drop`](#drop)
  - このアビリティを持つ型の値をポップ/ドロップすることを許可します。
- [`store`](#store)
  - このアビリティを持つ型の値がストレージ内の値の中に存在することを許可します。
  - Suiでは、`store`は[オブジェクト](./abilities/object)内に保存できるデータを制御します。
    `store`は、定義モジュールの外部に転送できる型も制御します。
- [`key`](#key)
  - 型がストレージの「キー」として機能することを許可します。これは、値がストレージ内の
    トップレベル値になることができることを意味します。つまり、ストレージに存在するために
    他の値に含まれる必要がありません。
  - Suiでは、`key`は[オブジェクト](./abilities/object)を示すために使用されます。

### `copy`

`copy`アビリティは、そのアビリティを持つ型の値をコピーすることを許可します。これは、[`copy`](./variables#move-and-copy)演算子を使用してローカル変数から値をコピーする機能と、[参照解決`*e`](./primitive-types/references#reading-and-writing-through-references)を通じて参照経由で値をコピーする機能を制御します。

値が`copy`を持つ場合、その値の内部に含まれるすべての値も`copy`を持ちます。

### `drop`

`drop`アビリティは、そのアビリティを持つ型の値をドロップすることを許可します。ドロップされるとは、その値が転送されず、Moveプログラムの実行中に事実上破棄されることを意味します。そのため、このアビリティは、以下を含む様々な場所で値を無視する機能を制御します：

- ローカル変数やパラメータで値を使用しない
- [`;`によるシーケンス](./variables#expression-blocks)で値を使用しない
- [代入](./variables#assignments)で変数の値を上書きする
- [`*e1 = e2`の書き込み](./primitive-types/references#reading-and-writing-through-references)時に参照経由で値を上書きする

値が`drop`を持つ場合、その値の内部に含まれるすべての値も`drop`を持ちます。

### `store`

`store`アビリティは、このアビリティを持つ型の値がストレージ内の値の中に存在することを許可しますが、必ずしもストレージ内のトップレベル値である必要はありません。これは、操作を直接制御しない唯一のアビリティです。代わりに、`key`と組み合わせて使用される際に、ストレージ内での存在を制御します。

値が`store`を持つ場合、その値の内部に含まれるすべての値も`store`を持ちます。

Suiでは、`store`は二重の役割を果たします。[オブジェクト](/storage/store-ability)内に表示できる値を制御し、また定義モジュールの外部に[転送](./abilities/object#transfer-rules)できるオブジェクトを制御します。

### `key`

`key`アビリティは、Moveのデプロイメントによって定義されるストレージ操作のキーとして型が機能することを許可します。これはMoveインスタンスごとに固有ですが、すべてのストレージ操作を制御するため、型がストレージプリミティブと一緒に使用されるには、その型が`key`アビリティを持つ必要があります。

値が`key`を持つ場合、その値の内部に含まれるすべての値は`store`を持ちます。これは、このような非対称性を持つ唯一のアビリティです。

Suiでは、`key`は[オブジェクト](./abilities/object)を示すために使用されます。

## 組み込み型

すべてのプリミティブな組み込み型は`copy`、`drop`、`store`を持ちます。

- `bool`、`u8`、`u16`、`u32`、`u64`、`u128`、`u256`、`address`はすべて`copy`、`drop`、
  `store`を持ちます。
- `vector<T>`は、`T`のアビリティに応じて`copy`、`drop`、`store`を持つ場合があります。
  - 詳細については[条件付きアビリティとジェネリック型](#conditional-abilities-and-generic-types)を参照してください。
- 不変参照`&`と可変参照`&mut`はどちらも`copy`と`drop`を持ちます。
  - これは参照自体のコピーとドロップを指し、参照が指すものではありません。
  - 参照はグローバルストレージに表示できないため、`store`を持ちません。

プリミティブ型はどれも`key`を持たないことに注意してください。つまり、どれもストレージ操作で直接使用することはできません。

## 構造体と列挙型への注釈

`struct`や`enum`がアビリティを持つことを宣言するには、データ型名の後でフィールド/バリアントの前または後に`has <ability>`で宣言します。例えば：

```move file=packages/reference/sources/abilities.move anchor=annotating_datatypes

```

この場合：`Ignorable*`は`drop`アビリティを持ちます。`Pair*`と`MyVec*`はどちらも`copy`、`drop`、`store`を持ちます。

これらのアビリティはすべて、これらの制限された操作に対して強力な保証を持ちます。操作は、値がそのアビリティを持つ場合にのみ実行できます。値が他のコレクションの深い部分にネストされていても同様です！

そのため：構造体のアビリティを宣言する際、フィールドに特定の要件が課されます。すべてのフィールドはこれらの制約を満たす必要があります。これらのルールは、構造体が上記で示されたアビリティの到達可能性ルールを満たすために必要です。構造体がアビリティで宣言される場合...

- `copy`、すべてのフィールドが`copy`を持つ必要があります。
- `drop`、すべてのフィールドが`drop`を持つ必要があります。
- `store`、すべてのフィールドが`store`を持つ必要があります。
- `key`、すべてのフィールドが`store`を持つ必要があります。
  - `key`は現在、自分自身を要求しない唯一のアビリティです。

列挙型は`key`を除いてこれらのアビリティのいずれかを持つことができます。列挙型はストレージのトップレベル値（オブジェクト）になることができないため、`key`を持つことはできません。ただし、列挙型のバリアントのフィールドには、構造体のフィールドと同じルールが適用されます。特に、列挙型がアビリティで宣言される場合...

- `copy`、すべてのバリアントのすべてのフィールドが`copy`を持つ必要があります。
- `drop`、すべてのバリアントのすべてのフィールドが`drop`を持つ必要があります。
- `store`、すべてのバリアントのすべてのフィールドが`store`を持つ必要があります。
- `key`、前述のとおり列挙型では許可されません。

例えば：

```move
// アビリティを持たない構造体
public struct NoAbilities {}

public struct WantsCopy has copy {
    f: NoAbilities, // ERROR 'NoAbilities'は'copy'を持ちません
}

public enum WantsCopyEnum has copy {
    Variant1
    Variant2(NoAbilities), // ERROR 'NoAbilities'は'copy'を持ちません
}
```

同様に：

```move
// アビリティを持たない構造体
public struct NoAbilities {}

public struct MyData has key {
    f: NoAbilities, // Error 'NoAbilities'は'store'を持ちません
}

public struct MyDataEnum has store {
    Variant1,
    Variant2(NoAbilities), // Error 'NoAbilities'は'store'を持ちません
}
```

## 条件付きアビリティとジェネリック型

ジェネリック型にアビリティが注釈されている場合、その型のすべてのインスタンスがそのアビリティを持つことが保証されるわけではありません。次の構造体宣言を考えてみましょう：

<!-- file=packages/reference/sources/abilities.move anchor=conditional_abilities -->

```move
public struct Cup<T> has copy, drop, store, key { item: T }
```

`Cup`がアビリティに関係なく任意の型を保持できれば非常に便利です。型システムは型パラメータを「見る」ことができるので、そのアビリティの保証に違反する型パラメータを「見た」場合、`Cup`からアビリティを削除できるはずです。

この動作は最初は少し混乱するかもしれませんが、コレクション型について考えるとより理解しやすくなります。組み込み型`vector`は次のような型宣言を持つと考えることができます：

```move
vector<T> has copy, drop, store;
```

私たちは`vector`が任意の型で動作することを望みます。異なるアビリティのために別々の`vector`型は欲しくありません。では、どのようなルールが欲しいでしょうか？上記のフィールドルールで欲しいものとまったく同じです。つまり、`vector`値をコピーするのは、内部要素がコピーできる場合にのみ安全です。`vector`値を無視するのは、内部要素が無視/ドロップできる場合にのみ安全です。そして、`vector`をストレージに置くのは、内部要素がストレージに置ける場合にのみ安全です。

この追加の表現力を持つために、型はその型のインスタンス化に応じて、宣言されたすべてのアビリティを持たない場合があります。代わりに、型が持つアビリティは、その宣言**と**型引数の両方に依存します。任意の型について、型パラメータは構造体内で使用されると悲観的に仮定されるため、型パラメータが上記のフィールドに対して記述された要件を満たす場合にのみアビリティが付与されます。上記の`Cup`を例に取ると：

- `Cup`は、`T`が`copy`を持つ場合にのみ`copy`アビリティを持ちます。
- `T`が`drop`を持つ場合にのみ`drop`を持ちます。
- `T`が`store`を持つ場合にのみ`store`を持ちます。
- `T`が`store`を持つ場合にのみ`key`を持ちます。

以下は、各アビリティに対するこの条件システムの例です：

### 例：条件付き`copy`

```move
public struct NoAbilities {}
public struct S has copy, drop { f: bool }
public struct Cup<T> has copy, drop, store { item: T }

fun example(c_x: Cup<u64>, c_s: Cup<S>) {
    // 有効、'u64'が'copy'を持つため'Cup<u64>'は'copy'を持ちます
    let c_x2 = copy c_x;
    // 有効、'S'が'copy'を持つため'Cup<S>'は'copy'を持ちます
    let c_s2 = copy c_s;
}

fun invalid(c_account: Cup<signer>, c_n: Cup<NoAbilities>) {
    // 無効、'Cup<signer>'は'copy'を持ちません。
    // 'Cup'はcopyで宣言されていても、'signer'が'copy'を持たないため
    // インスタンスは'copy'を持ちません
    let c_account2 = copy c_account;
    // 無効、'NoAbilities'が'copy'を持たないため
    // 'Cup<NoAbilities>'は'copy'を持ちません
    let c_n2 = copy c_n;
}
```

### 例：条件付き`drop`

```move
public struct NoAbilities {}
public struct S has copy, drop { f: bool }
public struct Cup<T> has copy, drop, store { item: T }

fun unused() {
    Cup<bool> { item: true }; // 有効、'Cup<bool>'は'drop'を持ちます
    Cup<S> { item: S { f: false }}; // 有効、'Cup<S>'は'drop'を持ちます
}

fun left_in_local(c_account: Cup<signer>): u64 {
    let c_b = Cup<bool> { item: true };
    let c_s = Cup<S> { item: S { f: false }};
    // 有効な戻り値：'c_account'、'c_b'、'c_s'は値を持ちますが
    // 'Cup<signer>'、'Cup<bool>'、'Cup<S>'は'drop'を持ちます
    0
}

fun invalid_unused() {
    // 無効、'Cup<NoAbilities>'は'drop'を持たないため無視できません。
    // 'Cup'は'drop'で宣言されていても、'NoAbilities'が'drop'を持たないため
    // インスタンスは'drop'を持ちません
    Cup<NoAbilities> { item: NoAbilities {} };
}

fun invalid_left_in_local(): u64 {
    let n = Cup<NoAbilities> { item: NoAbilities {} };
    // 無効な戻り値：'c_n'は値を持ちますが
    // 'Cup<NoAbilities>'は'drop'を持ちません
    0
}
```

### 例：条件付き`store`

```move
public struct Cup<T> has copy, drop, store { item: T }

// 'MyInnerData'は'store'で宣言されているため、すべてのフィールドが'store'を必要とします
struct MyInnerData has store {
    yes: Cup<u64>, // 有効、'Cup<u64>'は'store'を持ちます
    // no: Cup<signer>, 無効、'Cup<signer>'は'store'を持ちません
}

// 'MyData'は'key'で宣言されているため、すべてのフィールドが'store'を必要とします
struct MyData has key {
    yes: Cup<u64>, // 有効、'Cup<u64>'は'store'を持ちます
    inner: Cup<MyInnerData>, // 有効、'Cup<MyInnerData>'は'store'を持ちます
    // no: Cup<signer>, 無効、'Cup<signer>'は'store'を持ちません
}
```

### 例：条件付き`key`

```move
public struct NoAbilities {}
public struct MyData<T> has key { f: T }

fun valid(addr: address) acquires MyData {
    // 有効、'MyData<u64>'は'key'を持ちます
    transfer(addr, MyData<u64> { f: 0 });
}

fun invalid(addr: address) {
   // 無効、'MyData<NoAbilities>'は'key'を持ちません
   transfer(addr, MyData<NoAbilities> { f: NoAbilities {} })
   // 無効、'MyData<NoAbilities>'は'key'を持ちません
   borrow<NoAbilities>(addr);
   // 無効、'MyData<NoAbilities>'は'key'を持ちません
   borrow_mut<NoAbilities>(addr);
}

// モックストレージ操作
native public fun transfer<T: key>(addr: address, value: T);
```
