# モジュールのインポート

<!--
    TODO: より良い例を作成する：
        1. 一般的なモジュールのインポート
        2. メンバーのインポート
        3. 複数メンバーのインポート
        4. インポートのグループ化
        5. グループ用のSelfキーワード
-->

<!--

目標：
    - インポート構文の表示
    - ローカル依存関係
    - 外部依存関係
    - 他のパッケージからのモジュールインポート

 -->

Moveはモジュールのインポートを許可することで、高いモジュール性とコードの再利用を実現しています。同じパッケージ内のモジュールは互いにインポートでき、新しいパッケージは既存のパッケージに依存し、それらのモジュールも使用できます。このセクションでは、モジュールのインポートの基本と、独自のコードでそれらを使用する方法について説明します。

## モジュールのインポート

同じパッケージ内で定義されたモジュールは互いにインポートできます。`use`キーワードの後にモジュールパスが続き、これはパッケージアドレス（またはエイリアス）と`::`で区切られたモジュール名で構成されます。

```move title="File: sources/module_one.move" file=packages/samples/sources/move-basics/importing-modules.move anchor=module_one

```

同じパッケージ内で定義された別のモジュールは、`use`キーワードを使用して最初のモジュールをインポートできます。

```move title="File: sources/module_two.move" file=packages/samples/sources/move-basics/importing-modules-two.move anchor=module_two

```

> 注意：他のモジュールからインポートしたい任意のアイテム（構造体、関数、定数など）は、定義モジュールの外部でアクセス可能にするために`public`（または`public(package)` - [可視性修飾子](./visibility)を参照）キーワードでマークする必要があります。例えば、`module_one`の`Character`構造体と`new`関数は、`module_two`で使用できるようにパブリックにマークされています。

## メンバーのインポート

モジュールから特定のメンバーをインポートすることもできます。これは、モジュールから単一の関数または単一の型のみが必要な場合に便利です。構文はモジュールのインポートと同じですが、モジュールパスの後にメンバー名を追加します。

```move file=packages/samples/sources/move-basics/importing-modules-members.move anchor=members

```

## インポートのグループ化

インポートは中括弧`{}`を使用して単一の`use`ステートメントにグループ化できます。これにより、同じモジュールまたはパッケージから複数のメンバーをインポートする際に、よりクリーンで整理されたコードが可能になります。

```move file=packages/samples/sources/move-basics/importing-modules-grouped.move anchor=grouped

```

Moveでは関数名のインポートは一般的ではありません。これは、関数名が重複して混乱を招く可能性があるためです。推奨される実践は、モジュール全体をインポートし、モジュールパスを使用して関数にアクセスすることです。型には一意の名前があり、個別にインポートすべきです。

グループインポートでメンバーとモジュール自体の両方をインポートするには、`Self`キーワードを使用できます。`Self`キーワードはモジュール自体を参照し、モジュールとそのメンバーをインポートするために使用できます。

```move file=packages/samples/sources/move-basics/importing-modules-self.move anchor=self

```

## 名前の競合の解決

異なるモジュールから複数のメンバーをインポートする際、名前の競合が発生する可能性があります。例えば、同じ名前の関数を持つ2つのモジュールをインポートした場合、関数にアクセスするためにモジュールパスを使用する必要があります。異なるパッケージに同じ名前のモジュールがある場合もあります。競合を解決し、曖昧さを避けるために、Moveはインポートされたメンバーをリネームする`as`キーワードを提供します。

```move file=packages/samples/sources/move-basics/importing-modules-conflict-resolution.move anchor=conflict

```

## 外部依存関係の追加

Moveパッケージは他のパッケージに依存できます。依存関係は`Move.toml`という[パッケージマニフェスト](./../concepts/manifest)ファイルにリストされます。

パッケージ依存関係は[パッケージマニフェスト](./../concepts/manifest)で以下のように定義されます：

```ini title="Move.toml"
[dependencies]
Example = { git = "https://github.com/Example/example.git", subdir = "path/to/package", rev = "v1.2.3" }
Local = { local = "../my_other_package" }
```

`dependencies`セクションには、各パッケージ依存関係のエントリが含まれています。エントリのキーはパッケージ名（例では`Example`または`Local`）で、値はgitインポートテーブルまたはローカルパスです。gitインポートには、パッケージのURL、パッケージが配置されているサブディレクトリ、パッケージのリビジョンが含まれます。ローカルパスは、パッケージディレクトリへの相対パスです。

依存関係を追加すると、そのすべての依存関係もパッケージで利用可能になります。

`Move.toml`ファイルに依存関係が追加されると、コンパイラはパッケージをビルドする際に自動的に依存関係を取得（および後で再取得）します。

> sui CLIのバージョン1.45から、システムパッケージは`Move.toml`に存在しない場合、すべてのパッケージの依存関係として自動的に含まれます。したがって、`MoveStdlib`、`Sui`、`System`、`Bridge`、`Deepbook`はすべて明示的なインポートなしで利用可能です。

## 他のパッケージからのモジュールインポート

通常、パッケージは`[addresses]`セクションでアドレスを定義します。完全なアドレスの代わりにエイリアスを使用できます。例えば、Suiの`coin`モジュールを参照するために`0x2::coin`を使用する代わりに、`sui::coin`を使用できます。`sui`エイリアスはSui Frameworkパッケージのマニフェストで定義されています。同様に、`std`エイリアスは標準ライブラリパッケージで定義されており、標準ライブラリモジュールにアクセスするために`0x1`の代わりに使用できます。

他のパッケージからモジュールをインポートするには、`use`キーワードの後にモジュールパスを使用します。モジュールパスは、パッケージアドレス（またはエイリアス）と`::`で区切られたモジュール名で構成されます。

```move file=packages/samples/sources/move-basics/importing-modules-external.move anchor=external

```

> 注意：モジュールアドレス名は、マニフェストファイル（`Move.toml`）の`[addresses]`セクションから来ます。`[dependencies]`セクションで使用される名前ではありません。