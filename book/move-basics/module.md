# モジュール

<!--

Chapter: Base Syntax
Goal: Introduce module keyword.
Notes:
    - modules are the base unit of code organization
    - module members are private by default
    - types internal to the module have special access rules
    - only module can pack and unpack its types

 -->

モジュールはMoveにおけるコード組織の基本単位です。モジュールはコードをグループ化して分離するために使用され、モジュールのすべてのメンバーはデフォルトでモジュールプライベートです。このセクションでは、モジュールの定義方法、メンバーの宣言方法、および他のモジュールからのアクセス方法を学びます。

## モジュール宣言

モジュールは`module`キーワードの後にパッケージアドレス、モジュール名、セミコロン、モジュール本体を続けて宣言します。モジュール名は`snake_case`（すべて小文字で単語間にアンダースコア）である必要があります。モジュール名はパッケージ内で一意である必要があります。

通常、`sources/`フォルダ内の単一ファイルには単一のモジュールが含まれます。ファイル名はモジュール名と一致する必要があります。例えば、`donut_shop`モジュールは`donut_shop.move`ファイルに保存されるべきです。コーディング規約について詳しくは、
[コーディング規約](./../guides/code-quality-checklist)セクションをご覧ください。

> ファイル内で複数のモジュールを宣言する必要がある場合は、[モジュールブロック](#module-block)構文を使用する必要があります。

```move file=packages/samples/sources/move-basics/module-label.move anchor=module

```

構造体、関数、定数、インポートはすべてモジュールの一部です：

- [構造体](./struct)
- [関数](./function)
- [定数](./constants)
- [インポート](./importing-modules)
- [構造体メソッド](./struct-methods)

## アドレスと名前付きアドレス

モジュールアドレスは、アドレス_リテラル_（`@`プレフィックスは不要）または[パッケージマニフェスト](./../concepts/manifest)で指定された名前付きアドレスの両方で指定できます。以下の例では、`Move.toml`の`[addresses]`セクションに`book = "0x0"`レコードがあるため、両者は同一です。

```move file=packages/samples/sources/move-basics/module.move anchor=address_literal

```

Move.tomlのアドレスセクション：

```toml
# Move.toml
[addresses]
book = "0x0"
```

## モジュールメンバー

モジュールメンバーはモジュール本体の内部で宣言されます。これを説明するために、構造体、関数、定数を持つシンプルなモジュールを定義してみましょう：

```move file=packages/samples/sources/move-basics/module-members.move anchor=members

```

## Module Block

The pre-2024 edition of Move required the body of the module to be a _module block_ - the contents
of the module needed to be surrounded by curly braces `{}`. The main reason to use block syntax and
not _label_ syntax is if you need to define more than one module in a file. However, using module
blocks is not recommended practice.

```move file=packages/samples/sources/move-basics/module.move anchor=members

```

## Further Reading

- [Modules](./../../reference/modules) in the Move Reference.
