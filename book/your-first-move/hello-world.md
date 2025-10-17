# Hello, World!

この章では、新しいパッケージの作成、シンプルなモジュールの記述、コンパイル、およびMove CLIでのテストの実行方法を学習します。[Suiをインストール](./../before-we-begin/install-sui.md)し、[IDE環境](./../before-we-begin/ide-support.md)を設定していることを確認してください。以下のコマンドを実行して、Suiが正しくインストールされているかテストしてください。

```bash
# クライアントのバージョンが表示されます。例：sui-client 1.22.0-036299745.
sui client --version
```

> Move CLIは、Move言語のコマンドラインインターフェースです。Suiバイナリに組み込まれており、パッケージの管理、コードのコンパイル、テストを行うためのコマンドセットを提供します。

この章の構成は以下の通りです：

- [新しいパッケージの作成](#create-a-new-package)
- [ディレクトリ構造](#directory-structure)
- [パッケージのコンパイル](#compiling-the-package)
- [テストの実行](#running-tests)

## 新しいパッケージの作成

新しいプログラムを作成するために、`sui move new` コマンドの後にアプリケーションの名前を続けて使用します。最初のプログラムは `hello_world` と呼びます。

> 注意：この章や他の章で、`$`（ドル記号）で始まる行を含むコードブロックを見つけた場合、それは続くコマンドがターミナルで実行されるべきことを意味します。記号自体は含めるべきではありません。これは、ターミナル環境でコマンドを表示する一般的な方法です。

```bash
$ sui move new hello_world
```

`sui move` コマンドは、Move CLI（組み込みコンパイラ、テストランナー、およびMoveに関するすべてのユーティリティ）へのアクセスを提供します。`new` コマンドの後にパッケージ名を続けると、新しいフォルダに新しいパッケージが作成されます。この場合、フォルダ名は「hello_world」です。

フォルダの内容を表示して、パッケージが正常に作成されたことを確認できます。

```bash
$ ls -l hello_world
Move.toml
sources
tests
```

## ディレクトリ構造

Move CLIは、アプリケーションの枠組みを作成し、ディレクトリ構造とすべての必要なファイルを事前に作成します。中身を見てみましょう。

```plaintext
hello_world
├── Move.toml
├── sources
│   └── hello_world.move
└── tests
    └── hello_world_tests.move
```

### マニフェスト

[パッケージマニフェスト](./../concepts/manifest.md)として知られる`Move.toml`ファイルには、パッケージの定義と設定が含まれています。これは、パッケージメタデータの管理、依存関係の取得、名前付きアドレスの登録のためにMove Compilerによって使用されます。詳細については[概念](./../concepts/index.md)の章で説明します。

> デフォルトでは、パッケージには1つの名前付きアドレス（パッケージの名前）があります。

```toml
[addresses]
hello_world = "0x0"
```

### ソース

`sources/`ディレクトリにはソースファイルが含まれています。Moveのソースファイルは_.move_拡張子を持ち、通常はファイル内で定義されるモジュールの名前が付けられます。例えば、この場合、ファイル名は_hello_world.move_で、Move CLIはすでにコメントアウトされたコードを配置しています：

```move
/*
/// Module: hello_world
module hello_world::hello_world;
*/
```

> `/*`と`*/`は、Moveのコメント区切り文字です。その間のすべてはコンパイラによって無視され、ドキュメントやメモに使用できます。コードにコメントを付けるすべての方法については、[基本構文](./../move-basics/comments.md)で説明します。

コメントアウトされたコードはモジュール定義で、キーワード`module`の後に名前付きアドレス（またはアドレスリテラル）とモジュール名が続きます。モジュール名はモジュールの一意の識別子であり、パッケージ内で一意である必要があります。モジュール名は、他のモジュールやトランザクションからモジュールを参照するために使用されます。

<!-- And the module name has to be a valid Move identifier: alphanumeric with underscores to separate words. A common convention is to call modules (and functions) in snake_case - all lowercase, with underscores. Coding conventions are important for readability and maintainability of the code, we summarize them in the Coding Conventions section. -->

### テスト

`tests/`ディレクトリにはパッケージのテストが含まれています。コンパイラは、通常のビルドプロセスではこれらのファイルを除外しますが、_test_および_dev_モードでは使用します。テストはMoveで記述され、`#[test]`属性でマークされます。テストは、別のモジュールにグループ化するか（通常は_module_name_tests.move_と呼ばれる）、テスト対象のモジュール内に配置できます。

モジュール、インポート、定数、関数には`#[test_only]`で注釈できます。この属性は、ビルドプロセスからモジュール、関数、またはインポートを除外するために使用されます。これは、チェーン上で公開されるコードに含めることなく、テスト用のヘルパーを追加したい場合に便利です。

_hello_world_tests.move_ファイルには、コメントアウトされたテストモジュールのテンプレートが含まれています：

```move
/*
#[test_only]
module hello_world::hello_world_tests;
// モジュールをインポートするためにこの行のコメントを外してください
// use hello_world::hello_world;

const ENotImplemented: u64 = 0;

#[test]
fun test_hello_world() {
    // pass
}

#[test, expected_failure(abort_code = hello_world::hello_world_tests::ENotImplemented)]
fun test_hello_world_fail() {
    abort ENotImplemented
}
*/
```

### その他のフォルダ

さらに、Move CLIは`examples/`フォルダをサポートしています。そこのファイルは`tests/`フォルダに配置されたファイルと同様に扱われます - _test_および_dev_モードでのみビルドされます。これらは、パッケージの使用方法や他のパッケージとの統合方法の例です。最も人気のある使用例は、ドキュメントの目的とライブラリパッケージです。

## パッケージのコンパイル

Moveはコンパイル言語であり、そのためソースファイルをMoveバイトコードにコンパイルする必要があります。バイトコードには、モジュール、そのメンバー、型に関する必要な情報のみが含まれ、コメントや一部の識別子（例えば定数の識別子）は除外されます。

これらの機能を示すために、_sources/hello_world.move_ファイルの内容を以下に置き換えてみましょう：

```move file=packages/hello_world/sources/hello_world.move anchor=source
```

コンパイル中にコードはビルドされますが、実行はされません。コンパイルされたパッケージには、他のモジュールやトランザクションから呼び出すことができる関数のみが含まれます。これらの概念については[概念](./../concepts/index.md)の章で説明します。しかし今は、_sui move build_を実行したときに何が起こるかを見てみましょう。

```bash
# `hello_world`フォルダから実行
$ sui move build

# または、フォルダに`cd`しなかった場合
$ sui move build --path hello_world
```

コンソールに以下のメッセージが出力されるはずです。

```plaintext
UPDATING GIT DEPENDENCY https://github.com/MystenLabs/sui.git
INCLUDING DEPENDENCY Bridge
INCLUDING DEPENDENCY DeepBook
INCLUDING DEPENDENCY SuiSystem
INCLUDING DEPENDENCY Sui
INCLUDING DEPENDENCY MoveStdlib
BUILDING hello_world
```

コンパイル中に、Move Compilerは自動的にbuildフォルダを作成し、取得してコンパイルしたすべての依存関係と、現在のパッケージのモジュールのバイトコードを配置します。

> Gitなどのバージョン管理システムを使用している場合、buildフォルダは無視されるべきです。例えば、`.gitignore`ファイルを使用して`build`を追加する必要があります。

## テストの実行

テストに入る前に、テストを追加する必要があります。Move Compilerは、Moveで書かれたテストをサポートし、実行環境を提供します。テストは、ソースファイルと`tests/`フォルダの両方に配置できます。テストは`#[test]`属性でマークされ、コンパイラによって自動的に発見されます。テストについては、[テスト](./../move-basics/testing.md)セクションで詳しく説明します。

`tests/hello_world_tests.move`の内容を以下の内容に置き換えてください：

```move file=packages/hello_world/tests/hello_world_tests.move anchor=test
```

ここでは`hello_world`モジュールをインポートし、その`hello_world`関数を呼び出して、出力が実際に文字列「Hello, World!」であることをテストします。テストが準備できたので、テストモードでパッケージをコンパイルし、テストを実行しましょう。Move CLIにはこのための`test`コマンドがあります：

```bash
$ sui move test
```

出力は以下のようになるはずです：

```plaintext
INCLUDING DEPENDENCY Bridge
INCLUDING DEPENDENCY DeepBook
INCLUDING DEPENDENCY SuiSystem
INCLUDING DEPENDENCY Sui
INCLUDING DEPENDENCY MoveStdlib
BUILDING hello_world
Running Move unit tests
[ PASS    ] 0x0::hello_world_tests::test_hello_world
Test result: OK. Total tests: 1; passed: 1; failed: 0
```

パッケージフォルダの外でテストを実行している場合は、パッケージへのパスを指定できます：

```bash
$ sui move test --path hello_world
```

文字列を指定することで、単一または複数のテストを一度に実行することもできます。その文字列を含むすべてのテスト名が実行されます：

```bash
$ sui move test test_hello
```

## 次のステップ

このセクションでは、Moveパッケージの基本について説明しました：その構造、マニフェスト、ビルド、およびテストフローについてです。[次のページ](./hello-sui)では、アプリケーションを書いて、コードがどのように構造化されているか、そして言語が何をできるかを見ていきます。

## 参考資料

- [パッケージマニフェスト](./../concepts/manifest.md) セクション
- [The Move Reference](./../../reference/packages)のパッケージ
