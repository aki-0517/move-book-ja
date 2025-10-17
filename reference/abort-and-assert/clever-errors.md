---
title: 'Clever Errors | リファレンス'
description:
  Clever errorsは、アサーションが失敗した場合やabortが発生した場合により情報豊富なエラーメッセージを可能にする機能です
---

# Clever Errors

Clever errorsは、アサーションが失敗した場合やabortが発生した場合により情報豊富なエラーメッセージを可能にする機能です。これらはソース機能であり、clever errorコードとclever error定数が宣言されたモジュールが与えられた場合に、行番号、定数名、定数値にアクセスするために必要な情報を含む`u64` abortコード値にコンパイルされます。このコンパイルのため、`u64` abortコード値から人間が読めるエラーメッセージに変換するには後処理が必要です。後処理はSui GraphQLサーバーとSui CLIによって自動的に実行されます。clever abortコードを手動でデコードしたい場合は、[Clever Abortコードの展開](#inflating-clever-abort-codes)で概説されたプロセスを使用できます。

> Clever errorsには他のデータの中でもソース行情報が含まれます。このため、ソースファイルの変更（例：自動フォーマット、新しいモジュールメンバーの追加、改行の追加）により、その値が変更される可能性があります。

## Clever Abortコード

Clever abortコードにより、定数が`#[error]`属性で注釈されている限り、非u64定数をabortコードとして使用できます。これらはアサーションと`abort`のコードの両方で使用できます。

```move
module 0x42::a_module;

#[error]
const EIsThree: vector<u8> = b"The value is three";

// `x`が3の場合、`EIsThree`でabortします
public fun double_except_three(x: u64): u64 {
    assert!(x != 3, EIsThree);
    x * x
}

// 常に`EIsThree`でabortします
public fun clever_abort() {
    abort EIsThree
}
```

この例では、`EIsThree`定数は`vector<u8>`であり、`u64`ではありません。しかし、`#[error]`属性により、定数をabortコードとして使用でき、実行時に以下を保持する`u64` abortコード値を生成します：

1. abortコードがclever abortコードであることを示す設定されたタグビット。
2. ソースファイルでabortが発生した行番号（例：7）。
3. 定数名のモジュールの識別子テーブル内のインデックス（例：`EIsThree`）。
4. モジュールの定数テーブル内の定数値のインデックス（例：`b"The value is three"`）。

16進数では、`double_except_three(3)`が呼び出された場合、以下の`u64` abortコードでabortします：

```
0x8000_0007_0001_0000
  ^       ^    ^    ^
  |       |    |    |
  |       |    |    |
  |       |    |    +-- 定数値インデックス = 0 (b"The value is three")
  |       |    +-- 定数名インデックス = 1 (EIsThree)
  |       +-- 行番号 = 7 (アサーションの行)
  +-- タグビット = 0b1000_0000_0000_0000
```

そして、人間が読めるエラーメッセージとして以下のようにレンダリングできます（例）：

```
Error from '0x42::a_module::double_except_three' (line 7), abort 'EIsThree': "The value is three"
```

このメッセージの正確なフォーマットは、clever errorをデコードするために使用されるツールによって異なる場合がありますが、上記のような人間が読めるエラーメッセージを生成するために必要なすべての情報が、エラーが発生したモジュールと組み合わせて`u64` abortコードに存在します。

> Clever abortコード値は`vector<u8>`である必要は_ありません_ -- Moveの任意の有効な定数型にすることができます。

## Abortコードなしのアサーション

abortコードなしのアサーションと`abort`文は、ソース行番号から自動的にabortコードを導出し、定数名と定数値の情報がそれぞれ`0xffff`のセンチネル値で埋められたclever error形式でエンコードされます。例えば：

```move
module 0x42::a_module;

#[test]
fun assert_false(x: bool) {
    assert!(false);
}

#[test]
fun abort_no_code() {
    abort
}
```

これら両方は以下を保持する`u64` abortコード値を生成します：

1. abortコードがclever abortコードであることを示す設定されたタグビット。
2. ソースファイルでabortが発生した行番号（例：6）。
3. 定数名のモジュールの識別子テーブルへのインデックスのセンチネル値`0xffff`。
4. モジュールの定数テーブル内の定数値のインデックスのセンチネル値`0xffff`。

16進数では、`assert_false(3)`が呼び出された場合、以下の`u64` abortコードでabortします：

```
0x8000_0004_ffff_ffff
  ^       ^    ^    ^
  |       |    |    |
  |       |    |    |
  |       |    |    +-- 定数値インデックス = 0xffff (センチネル値)
  |       |    +-- 定数名インデックス = 0xffff (センチネル値)
  |       +-- 行番号 = 4 (アサーションの行)
  +-- タグビット = 0b1000_0000_0000_0000
```

## Clever Errorsとマクロ

clever abortコードの行番号情報は、abortが発生した場所のソースファイルから導出されます。特に、関数の場合は関数内の行番号になりますが、マクロの場合はマクロが呼び出された場所の行番号になります。これは、abort条件を発生させる可能性のあるマクロをユーザーが使用し、それでも有用なエラーメッセージを得られる方法を提供するため、マクロを書く際に非常に有用です。

```move
module 0x42::macro_exporter;

public macro fun assert_false() {
    assert!(false);
}

public macro fun abort_always() {
    abort
}

public fun assert_false_fun() {
    assert!(false); // 常にこの呼び出しの行番号でabortします
}

public fun abort_always_fun() {
    abort // 常にこの呼び出しの行番号でabortします
}
```

そして、これらのマクロを使用するモジュールでは：

```move
module 0x42::user_module;

use 0x42::macro_exporter::{
    assert_false,
    abort_always,
    assert_false_fun,
    abort_always_fun
};

fun invoke_assert_false() {
    assert_false!(); // この呼び出しの行番号でabortします
}

fun invoke_abort_always() {
    abort_always!(); // この呼び出しの行番号でabortします
}

fun invoke_assert_false_fun() {
    assert_false_fun(); // `assert_false_fun`内のアサーションの行番号でabortします
}

fun invoke_abort_always_fun() {
    abort_always_fun(); // `abort_always_fun`内の`abort`の行番号でabortします
}
```

## Clever Abortコードの展開

正確には、clever abortコードのレイアウトは以下の通りです：

```

|<tagbit>|<reserved>|<source line number>|<module identifier index>|<module constant index>|
+--------+----------+--------------------+-------------------------+-----------------------+
| 1-bit  | 15-bits  |       16-bits      |     16-bits             |        16-bits        |

```

Move abortには追加の情報が含まれることに注意してください -- 重要なのは、エラーが発生したモジュールです。これは、識別子インデックスと定数インデックスがモジュールの識別子テーブルと定数テーブル（センチネル値が設定されていない場合）に相対的であるため重要です。

> clever abortコードをデコードするには、識別子インデックスまたは定数インデックスのいずれかがセンチネル値`0xffff`に設定されていない場合、エラーが発生したモジュールを知る必要があります。

疑似コードでは、clever abortコードを以下のようにデコードできます：

```rust
// MoveAbortで利用可能な情報
let clever_abort_code: u64 = ...;
let (package_id, module_name): (PackageStorageId, ModuleName) = ...;

let is_clever_abort = (clever_abort_code & 0x8000_0000_0000_0000) != 0;

if is_clever_abort {
    // 行番号、識別子インデックス、定数インデックスを取得
    // 識別子インデックスと定数インデックスは'0xffff'に設定されている場合はセンチネル値
    let line_number = ((clever_abort_code & 0x0000_ffff_0000_0000) >> 32) as u16;
    let identifier_index = ((clever_abort_code & 0x0000_0000_ffff_0000) >> 16) as u16;
    let constant_index = ((clever_abort_code & 0x0000_0000_0000_ffff)) as u16;

    // 行エラーメッセージを出力
    print!("Error from '{}::{}' (line {})", package_id, module_name, line_number);

    // 両方がセンチネル値の場合は何も出力したりモジュールを読み込む必要はありません
    if identifier_index == 0xffff && constant_index == 0xffff {
        return;
    }

    // 定数名と値が0xffffでない場合にのみ必要
    let module: CompiledModule = fetch_module(package_id, module_name);

    // 定数名を出力（ある場合）
    if identifier_index != 0xffff {
        let constant_name = module.get_identifier_at_table_index(identifier_index);
        print!(", '{}'", constant_name);
    }

    // 定数値を出力（ある場合）
    if constant_index != 0xffff {
        let constant_value = module
            .get_constant_at_table_index(constant_index)
            .deserialize_on_constant_type()
            .to_string();

        print!(": {}", constant_value);
    }

    return;
}
```
