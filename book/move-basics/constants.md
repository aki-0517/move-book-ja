# 定数

<!--

Chapter: Basic Syntax
Goal: Introduce constants.
Notes:
    - constants are immutable
    - constants are private
    - start with a capital letter always
    - stored in the bytecode (but w/o a name)
    - mention standard for naming constants

Links:
    - next section (abort and assert)
    - coding conventions (constants)
    - constants (language reference)

 -->

定数はモジュールレベルで定義される不変の値です。定数は、モジュール全体で使用される静的値に名前を付ける方法としてしばしば使用されます。たとえば、製品のデフォルト価格がある場合、それに対して定数を定義するかもしれません。定数はモジュールのバイトコードに格納され、使用されるたびに値がコピーされます。

```move file=packages/samples/sources/move-basics/constants-shop-price.move anchor=shop_price

```

## 命名規約

定数は大文字で始まる必要があります - これはコンパイラレベルで強制されます。値として使用される定数の場合、すべて大文字と単語間のアンダースコアを使用するのが慣例で、これにより定数がコード内の他の識別子から際立つようになります。[エラー定数](./assert-and-abort#error-constants)はECamelCaseで書かれる例外です。

```move file=packages/samples/sources/move-basics/constants-naming.move anchor=naming

```

## 定数は不変

定数は変更されたり、新しい値が代入されたりすることはありません。パッケージバイトコードの一部として、本質的に不変です。

```move
module book::immutable_constants;

const ITEM_PRICE: u64 = 100;

// エラーが発生します
fun change_price() {
    ITEM_PRICE = 200;
}
```

## 設定パターンの使用

アプリケーションの一般的な用途は、コードベース全体で使用される定数のセットを定義することです。しかし、定数はモジュールプライベートであるため、他のモジュールからアクセスできません。これを解決する一つの方法は、定数をエクスポートする「config」モジュールを定義することです。

```move file=packages/samples/sources/move-basics/constants-config.move anchor=config

```

この方法で、他のモジュールが定数をインポートして読み取ることができ、更新プロセスが簡素化されます。定数を変更する必要がある場合、パッケージのアップグレード時にconfigモジュールのみを更新する必要があります。

## リンク

- Move Referenceの[Constants](./../../reference/constants)
- [定数のコーディング規約](./../guides/code-quality-checklist#regular-constant-are-all_caps)
