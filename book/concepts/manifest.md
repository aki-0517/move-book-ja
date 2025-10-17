# パッケージマニフェスト（Package Manifest）

`Move.toml`は[パッケージ](./packages)とその依存関係を記述するマニフェストファイルです。
[TOML](https://toml.io/en/)形式で記述され、複数のセクションが含まれており、最も重要なセクションは`[package]`、`[dependencies]`、`[addresses]`です。

```toml
[package]
name = "my_project"
version = "0.0.0"
edition = "2024"

[dependencies]
Example = { git = "https://github.com/example/example.git", subdir = "path/to/package", rev = "framework/testnet" }

[addresses]
std =  "0x1"
alice = "0xA11CE"

[dev-addresses]
alice = "0xB0B"
```

## セクション

### Package

`[package]`セクションはパッケージを記述するために使用されます。このセクションのフィールドはチェーン上に公開されませんが、ツールやリリース管理で使用され、コンパイラのMove版も指定します。

- `name` - インポート時のパッケージ名
- `version` - パッケージのバージョン、リリース管理で使用可能
- `edition` - Move言語の版。現在は`2024`のみが有効な値

<!-- published-at -->

### Dependencies

`[dependencies]`セクションはプロジェクトの依存関係を指定するために使用されます。各依存関係はキーと値のペアで指定され、キーは依存関係の名前、値は依存関係の仕様です。依存関係の仕様はgitリポジトリのURLまたはローカルディレクトリへのパスを指定できます。

```toml
# gitリポジトリ
Example = { git = "https://github.com/example/example.git", subdir = "path/to/package", rev = "framework/testnet" }

# ローカルディレクトリ
MyPackage = { local = "../my-package" }
```

パッケージは他のパッケージからアドレスもインポートします。例えば、Sui依存関係はプロジェクトに`std`と`sui`のアドレスを追加します。これらのアドレスはコード内でアドレスのエイリアスとして使用できます。

Sui CLI バージョン1.45以降では、Suiシステムパッケージ（`std`、`sui`、`system`、`bridge`、`deepbook`）が明示的にリストされていない場合、自動的に依存関係として追加されます。

### オーバーライドによるバージョン競合の解決

依存関係で同じパッケージの競合するバージョンがある場合があります。例えば、Exampleパッケージの異なるバージョンを使用する2つの依存関係がある場合、`[dependencies]`セクションで依存関係をオーバーライドできます。これを行うには、依存関係に`override`フィールドを追加します。`[dependencies]`セクションで指定された依存関係のバージョンが、依存関係自体で指定されたものの代わりに使用されます。

```toml
[dependencies]
Example = { override = true, git = "https://github.com/example/example.git", subdir = "crates/sui-framework/packages/sui-framework", rev = "framework/testnet" }
```

### Dev-dependencies

マニフェストに`[dev-dependencies]`セクションを追加することができます。これは開発およびテストモードで依存関係をオーバーライドするために使用されます。例えば、開発モードで異なるバージョンのSuiパッケージを使用したい場合、カスタム依存関係の仕様を`[dev-dependencies]`セクションに追加できます。

### Addresses

`[addresses]`セクションはアドレスのエイリアスを追加するために使用されます。このセクションで任意のアドレスを指定し、コード内でエイリアスとして使用できます。例えば、このセクションに`alice = "0xA11CE"`を追加すると、コード内で`alice`を`0xA11CE`として使用できます。

### Dev-addresses

`[dev-addresses]`セクションは`[addresses]`と同じですが、テストおよび開発モードでのみ動作します。重要な点は、このセクションで新しいエイリアスを導入することはできず、既存のもののみをオーバーライドできることです。上記の例で、このセクションに`alice = "0xB0B"`を追加すると、テストおよび開発モードでは`alice`アドレスが`0xB0B`となり、通常のビルドでは`0xA11CE`となります。

## TOMLスタイル

TOML形式はテーブルに対して、インライン形式と複数行形式の2つのスタイルをサポートしています。上記の例はインライン形式を使用していますが、複数行形式も使用できます。`[package]`セクションでは使用したくないかもしれませんが、依存関係には便利です。

```toml
# インライン形式
[dependencies]
Example = { override = true, git = "https://github.com/example/example.git", subdir = "crates/sui-framework/packages/sui-framework", rev = "framework/testnet" }
MyPackage = { local = "../my-package" }
```

```toml
# 複数行形式
[dependencies.Example]
override = true
git = "https://github.com/example/example.git"
subdir = "crates/sui-framework/packages/sui-framework"
rev = "framework/testnet"

[dependencies.MyPackage]
local = "../my-package"
```

## 関連資料

- Move リファレンスの[パッケージ](./../../reference/packages)
