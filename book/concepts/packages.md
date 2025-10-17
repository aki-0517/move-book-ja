# パッケージ（Package）

<!--

    - packages and how they're stored
        - overview of packages and their contents (use a diagram)
        - how a package is created, and what it consists of
        - what is the package manifest
        - describe how "name" field is used
        - mention the "edition" field
        - what are the folders in a package and what are they for
        - how packages are imported (give Sui as an example)
        - what are addresses, and how they identify packages
        - how packages are published
        - leave a note that packages are also *upgradable*

-->

Moveはスマートコントラクト（ブロックチェーン上に格納され実行されるプログラム）を書くための言語です。
1つのプログラムはパッケージに組織化されます。パッケージはブロックチェーン上に公開され、
[アドレス](./address)で識別されます。公開されたパッケージは、その関数を呼び出す
[トランザクション](./what-is-a-transaction)を送信することで操作できます。また、他のパッケージの依存関係としても機能できます。

> 新しいパッケージを作成するには、`sui move new`コマンドを使用します。コマンドについて詳しく知るには、
> `sui move new --help`を実行してください。

パッケージはモジュールで構成されます。モジュールは関数、型、その他のアイテムを含む個別のスコープです。

```
package 0x...
    module a
        struct A1
        fun hello_world()
    module b
        struct B1
        fun hello_package()
```

## パッケージ構造

ローカルでは、パッケージは`Move.toml`ファイルと`sources`ディレクトリを持つディレクトリです。`Move.toml`
ファイル（「パッケージマニフェスト」と呼ばれる）にはパッケージに関するメタデータが含まれ、`sources`
ディレクトリにはモジュールのソースコードが含まれます。パッケージは通常次のような構造になります：

```
sources/
    my_module.move
    another_module.move
    ...
tests/
    ...
examples/
    using_my_module.move
Move.toml
```

`tests`ディレクトリはオプションで、パッケージのテストが含まれます。`tests`
ディレクトリに配置されたコードはオンチェーンに公開されず、テストでのみ利用可能です。`examples`ディレクトリは
コード例に使用でき、これもオンチェーンに公開されません。

## 公開されたパッケージ

開発中は、パッケージにアドレスがなく、`0x0`に設定する必要があります。パッケージが公開されると、
ブロックチェーン上でモジュールのバイトコードを含む単一の一意の[アドレス](./address)を取得します。
公開されたパッケージは_不変_となり、トランザクションを送信することで操作できます。

```
0x...
    my_module: <bytecode>
    another_module: <bytecode>
```

## リンク

- [パッケージマニフェスト](./manifest)
- [アドレス](./address)
- Move リファレンスの[パッケージ](./../../reference/packages)
