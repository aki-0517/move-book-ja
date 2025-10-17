# Hello, Sui!

[前のセクション](./hello-world)では、新しいパッケージを作成し、Move パッケージの作成、ビルド、テストの基本的な流れを説明しました。このセクションでは、ストレージモデルを使用し、相互作用可能なシンプルなアプリケーションを作成します。そのために、シンプルなtodoリストアプリケーションを作成します。

## 新しいパッケージを作成する

[Hello, World!](./hello-world)と同じ流れに従って、`todo_list`という名前の新しいパッケージを作成します。

```bash
$ sui move new todo_list
```

## コードを追加する

作業を効率化し、アプリケーションロジックに集中するために、todoリストアプリケーションのコードを提供します。_sources/todo_list.move_ ファイルの内容を次のコードに置き換えてください：

> 注意：最初は内容が圧倒的に見えるかもしれませんが、以下のセクションで詳しく説明します。今は目の前のことに集中してみてください。

```move file=packages/todo_list/sources/todo_list.move anchor=all

```

## パッケージをビルドする

すべてが正しく行われていることを確認するために、`sui move build` コマンドを実行してパッケージをビルドしましょう。すべてが正しければ、次のような出力が表示されるはずです：

```bash
$ sui move build
UPDATING GIT DEPENDENCY https://github.com/MystenLabs/sui.git
INCLUDING DEPENDENCY Bridge
INCLUDING DEPENDENCY DeepBook
INCLUDING DEPENDENCY SuiSystem
INCLUDING DEPENDENCY Sui
INCLUDING DEPENDENCY MoveStdlib
BUILDING todo_list
```

この出力に続いてエラーがなければ、パッケージのビルドに成功しています。エラーがある場合は、以下を確認してください：

- コードが正しくコピーされている
- ファイル名とパッケージ名が正しい

この段階でコードが失敗する他の理由はそれほど多くありません。それでも問題が発生している場合は、
[この場所](https://github.com/MystenLabs/move-book/tree/main/packages/todo_list)でパッケージの構造を確認してみてください。

## アカウントをセットアップする

> 既にアカウントがセットアップされている場合は、この手順をスキップできます。

パッケージを公開し、相互作用するためには、アカウントをセットアップする必要があります。開発中は、独自の
[ローカルネットワーク](https://docs.sui.io/guides/developer/getting-started/local-network)を実行するのが最適です。現在は `RUST_LOG="off,sui_node=info" sui start --with-faucet --force-regenesis` を実行するだけです。Sui ローカルネットワークはマシンのポート9000で実行されるので、そのポートが他のアプリケーションで使用されていないことを確認してください。

初回の場合は、新しいアカウントを作成する必要があります。これを行うには、`sui client` コマンドを実行すると、CLIが複数の質問を表示します。回答は以下の `>` で示されています：

```bash
$ sui client
Config file ["/path/to/home/.sui/sui_config/client.yaml"] doesn't exist, do you want to connect to a Sui Full node server [y/N]?
> y
Sui Full node server URL (Defaults to Sui Testnet if not specified) :
> http://127.0.0.1:9000
Environment alias for [http://127.0.0.1:9000] :
> localnet
Select key scheme to generate keypair (0 for ed25519, 1 for secp256k1, 2: for secp256r1):
> 0
```

質問に答えた後、CLIは新しいキーペアを生成し、設定ファイルに保存します。これで、このアカウントを使用してネットワークと相互作用できます。

アカウントが正しくセットアップされていることを確認するには、`sui client active-address` コマンドを実行します：

```bash
$ sui client active-address
0x....
```

このコマンドは、あなたのアカウントのアドレスを出力します。これは `0x` で始まり、その後に64文字が続きます。

## コインを要求する

_devnet_ と _testnet_ 環境では、CLIはあなたのアカウントにコインを要求する方法を提供しているので、ネットワークと相互作用できます。コインを要求するには、`sui client faucet` コマンドを実行します：

```bash
$ sui client faucet
Request successful. It can take up to 1 minute to get the coin. Run sui client gas to check your gas coins.
```

少し待った後、`sui client balance` コマンドを実行して、Coinオブジェクトがあなたのアカウントに送信されたことを確認できます：

```bash
$ sui client balance
╭────────────────────────────────────────╮
│ Balance of coins owned by this address │
├────────────────────────────────────────┤
│ ╭──────────────────────────────────╮   │
│ │ coin  balance (raw)  balance     │   │
│ ├──────────────────────────────────┤   │
│ │ Sui   1000000000    1.00 SUI     │   │
│ ╰──────────────────────────────────╯   │
╰────────────────────────────────────────╯
```

あるいは、`sui client objects` コマンドを実行して、あなたのアカウントが所有している _オブジェクト_ を照会することもできます。オブジェクトIDは一意であり、digestも同様であるため、実際の出力は異なりますが、構造は類似しています：

```bash
$ sui client objects
╭───────────────────────────────────────────────────────────────────────────────────────╮
│ ╭────────────┬──────────────────────────────────────────────────────────────────────╮ │
│ │ objectId   │  0x4ea1303e4f5e2f65fc3709bc0fb70a3035fdd2d53dbcff33e026a50a742ce0de  │ │
│ │ version    │  4                                                                   │ │
│ │ digest     │  nA68oa8gab/CdIRw+240wze8u0P+sRe4vcisbENcR4U=                        │ │
│ │ objectType │  0x0000..0002::coin::Coin                                            │ │
│ ╰────────────┴──────────────────────────────────────────────────────────────────────╯ │
╰───────────────────────────────────────────────────────────────────────────────────────╯
```

これで、アカウントがセットアップされ、アカウントにコインが入ったので、ネットワークと相互作用できます。
まず、パッケージをネットワークに公開することから始めます。

## 公開

パッケージをネットワークに公開するために、`sui client publish` コマンドを使用します。このコマンドは自動的にパッケージをビルドし、そのバイトコードを使用して単一のトランザクションで公開します。

> 公開時に `--gas-budget` 引数を使用しています。これは、トランザクションにどれだけのgasを費やすかを指定します。このセクションではこのトピックに触れませんが、Suiのすべてのトランザクションにはgasが必要であり、gasはSUIコインで支払われることを知っておくことが重要です。
> `--gas-budget` は必須パラメータではないことも注目に値します。設定しない場合は、デフォルトの消費制限があります。

`gas-budget` は _MIST_ で指定されます。1 SUI は 10^9 MIST に等しいです。デモンストレーションのために、100,000,000 MIST（0.1 SUI）を使用します。

```bash
# `todo_list` フォルダから実行
$ sui client publish --gas-budget 100000000

# あるいは、パッケージへのパスを指定することもできます
$ sui client publish --gas-budget 100000000 todo_list
```

publishコマンドの出力はかなり長いので、部分に分けて表示し説明します。

```bash
$ sui client publish --gas-budget 100000000
UPDATING GIT DEPENDENCY https://github.com/MystenLabs/sui.git
INCLUDING DEPENDENCY Bridge
INCLUDING DEPENDENCY DeepBook
INCLUDING DEPENDENCY SuiSystem
INCLUDING DEPENDENCY Sui
INCLUDING DEPENDENCY MoveStdlib
BUILDING todo_list
Successfully verified dependencies on-chain against source.
Transaction Digest: GpcDV6JjjGQMRwHpEz582qsd5MpCYgSwrDAq1JXcpFjW
```

ご覧のように、`publish` コマンドを実行すると、CLIはまずパッケージをビルドし、次にオンチェーンの依存関係を検証し、最後にパッケージを公開します。コマンドの出力はトランザクションダイジェストで、これはトランザクションの一意識別子であり、トランザクションステータスの照会に使用できます。

### トランザクションデータ

`TransactionData` というタイトルのセクションには、送信したトランザクションに関する情報が含まれています。
あなたのアドレスである `sender`、`--gas-budget` 引数で設定された `gas_budget`、支払いに使用したCoinなどのフィールドが含まれています。また、CLIによって実行されたコマンドも出力されます。この例では、`Publish` と `TransferObject` コマンドが実行されました。後者は特別なオブジェクト `UpgradeCap` を送信者に転送します。

```plaintext
╭──────────────────────────────────────────────────────────────────────────────────────────────────────────────╮
│ Transaction Data                                                                                             │
├──────────────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ Sender: 0x091ef55506ad814920adcef32045f9078f2f6e9a72f4cf253a1e6274157380a1                                   │
│ Gas Owner: 0x091ef55506ad814920adcef32045f9078f2f6e9a72f4cf253a1e6274157380a1                                │
│ Gas Budget: 100000000 MIST                                                                                   │
│ Gas Price: 1000 MIST                                                                                         │
│ Gas Payment:                                                                                                 │
│  ┌──                                                                                                         │
│  │ ID: 0x4ea1303e4f5e2f65fc3709bc0fb70a3035fdd2d53dbcff33e026a50a742ce0de                                    │
│  │ Version: 7                                                                                                │
│  │ Digest: AXYPnups8A5J6pkvLa6RekX2ye3qur66EZ88mEbaUDQ1                                                      │
│  └──                                                                                                         │
│                                                                                                              │
│ Transaction Kind: Programmable                                                                               │
│ ╭──────────────────────────────────────────────────────────────────────────────────────────────────────────╮ │
│ │ Input Objects                                                                                            │ │
│ ├──────────────────────────────────────────────────────────────────────────────────────────────────────────┤ │
│ │ 0   Pure Arg: Type: address, Value: "0x091ef55506ad814920adcef32045f9078f2f6e9a72f4cf253a1e6274157380a1" │ │
│ ╰──────────────────────────────────────────────────────────────────────────────────────────────────────────╯ │
│ ╭─────────────────────────────────────────────────────────────────────────╮                                  │
│ │ Commands                                                                │                                  │
│ ├─────────────────────────────────────────────────────────────────────────┤                                  │
│ │ 0  Publish:                                                             │                                  │
│ │  ┌                                                                      │                                  │
│ │  │ Dependencies:                                                        │                                  │
│ │  │   0x0000000000000000000000000000000000000000000000000000000000000001 │                                  │
│ │  │   0x0000000000000000000000000000000000000000000000000000000000000002 │                                  │
│ │  └                                                                      │                                  │
│ │                                                                         │                                  │
│ │ 1  TransferObjects:                                                     │                                  │
│ │  ┌                                                                      │                                  │
│ │  │ Arguments:                                                           │                                  │
│ │  │   Result 0                                                           │                                  │
│ │  │ Address: Input  0                                                    │                                  │
│ │  └                                                                      │                                  │
│ ╰─────────────────────────────────────────────────────────────────────────╯                                  │
│                                                                                                              │
│ Signatures:                                                                                                  │
│    gebjSbVwZwTkizfYg2XIuzdx+d66VxFz8EmVaisVFiV3GkDay6L+hQG3n2CQ1hrWphP6ZLc7bd1WRq4ss+hQAQ==                  │
│                                                                                                              │
╰──────────────────────────────────────────────────────────────────────────────────────────────────────────────╯
```

### トランザクション効果

トランザクション効果には、トランザクションのステータス、トランザクションがネットワークの状態に加えた変更、およびトランザクションに関与したオブジェクトが含まれています。

```plaintext
╭───────────────────────────────────────────────────────────────────────────────────────────────────╮
│ Transaction Effects                                                                               │
├───────────────────────────────────────────────────────────────────────────────────────────────────┤
│ Digest: GpcDV6JjjGQMRwHpEz582qsd5MpCYgSwrDAq1JXcpFjW                                              │
│ Status: Success                                                                                   │
│ Executed Epoch: 411                                                                               │
│                                                                                                   │
│ Created Objects:                                                                                  │
│  ┌──                                                                                              │
│  │ ID: 0x160f7856e13b27e5a025112f361370f4efc2c2659cb0023f1e99a8a84d1652f3                         │
│  │ Owner: Account Address ( 0x091ef55506ad814920adcef32045f9078f2f6e9a72f4cf253a1e6274157380a1 )  │
│  │ Version: 8                                                                                     │
│  │ Digest: 8y6bhwvQrGJHDckUZmj2HDAjfkyVqHohhvY1Fvzyj7ec                                           │
│  └──                                                                                              │
│  ┌──                                                                                              │
│  │ ID: 0x468daa33dfcb3e17162bbc8928f6ec73744bb08d838d1b6eb94eac99269b29fe                         │
│  │ Owner: Immutable                                                                               │
│  │ Version: 1                                                                                     │
│  │ Digest: Ein91NF2hc3qC4XYoMUFMfin9U23xQmDAdEMSHLae7MK                                           │
│  └──                                                                                              │
│ Mutated Objects:                                                                                  │
│  ┌──                                                                                              │
│  │ ID: 0x4ea1303e4f5e2f65fc3709bc0fb70a3035fdd2d53dbcff33e026a50a742ce0de                         │
│  │ Owner: Account Address ( 0x091ef55506ad814920adcef32045f9078f2f6e9a72f4cf253a1e6274157380a1 )  │
│  │ Version: 8                                                                                     │
│  │ Digest: 7ydahjaM47Gyb33PB4qnW2ZAGqZvDuWScV6sWPiv7LTc                                           │
│  └──                                                                                              │
│ Gas Object:                                                                                       │
│  ┌──                                                                                              │
│  │ ID: 0x4ea1303e4f5e2f65fc3709bc0fb70a3035fdd2d53dbcff33e026a50a742ce0de                         │
│  │ Owner: Account Address ( 0x091ef55506ad814920adcef32045f9078f2f6e9a72f4cf253a1e6274157380a1 )  │
│  │ Version: 8                                                                                     │
│  │ Digest: 7ydahjaM47Gyb33PB4qnW2ZAGqZvDuWScV6sWPiv7LTc                                           │
│  └──                                                                                              │
│ Gas Cost Summary:                                                                                 │
│    Storage Cost: 10404400 MIST                                                                    │
│    Computation Cost: 1000000 MIST                                                                 │
│    Storage Rebate: 978120 MIST                                                                    │
│    Non-refundable Storage Fee: 9880 MIST                                                          │
│                                                                                                   │
│ Transaction Dependencies:                                                                         │
│    7Ukrc5GqdFqTA41wvWgreCdHn2vRLfgQ3YMFkdks72Vk                                                   │
│    7d4amuHGhjtYKujEs9YkJARzNEn4mRbWWv3fn4cdKdyh                                                   │
╰───────────────────────────────────────────────────────────────────────────────────────────────────╯
```

### イベント

_イベント_ が発行されていれば、このセクションで確認できます。私たちのパッケージはイベントを使用していないので、このセクションは空です。

```plaintext
╭─────────────────────────────╮
│ No transaction block events │
╰─────────────────────────────╯
```

### オブジェクトの変更

これらは、トランザクションが _オブジェクト_ に加えた変更です。私たちの場合、送信者が将来パッケージをアップグレードできる特別なオブジェクトである新しい `UpgradeCap` オブジェクトを _作成_ し、Gasオブジェクトを _変更_ し、新しいパッケージを _公開_ しました。パッケージもSuiではオブジェクトです。

```plaintext
╭──────────────────────────────────────────────────────────────────────────────────────────────────╮
│ Object Changes                                                                                   │
├──────────────────────────────────────────────────────────────────────────────────────────────────┤
│ Created Objects:                                                                                 │
│  ┌──                                                                                             │
│  │ ObjectID: 0x160f7856e13b27e5a025112f361370f4efc2c2659cb0023f1e99a8a84d1652f3                  │
│  │ Sender: 0x091ef55506ad814920adcef32045f9078f2f6e9a72f4cf253a1e6274157380a1                    │
│  │ Owner: Account Address ( 0x091ef55506ad814920adcef32045f9078f2f6e9a72f4cf253a1e6274157380a1 ) │
│  │ ObjectType: 0x2::package::UpgradeCap                                                          │
│  │ Version: 8                                                                                    │
│  │ Digest: 8y6bhwvQrGJHDckUZmj2HDAjfkyVqHohhvY1Fvzyj7ec                                          │
│  └──                                                                                             │
│ Mutated Objects:                                                                                 │
│  ┌──                                                                                             │
│  │ ObjectID: 0x4ea1303e4f5e2f65fc3709bc0fb70a3035fdd2d53dbcff33e026a50a742ce0de                  │
│  │ Sender: 0x091ef55506ad814920adcef32045f9078f2f6e9a72f4cf253a1e6274157380a1                    │
│  │ Owner: Account Address ( 0x091ef55506ad814920adcef32045f9078f2f6e9a72f4cf253a1e6274157380a1 ) │
│  │ ObjectType: 0x2::coin::Coin<0x2::sui::SUI>                                                    │
│  │ Version: 8                                                                                    │
│  │ Digest: 7ydahjaM47Gyb33PB4qnW2ZAGqZvDuWScV6sWPiv7LTc                                          │
│  └──                                                                                             │
│ Published Objects:                                                                               │
│  ┌──                                                                                             │
│  │ PackageID: 0x468daa33dfcb3e17162bbc8928f6ec73744bb08d838d1b6eb94eac99269b29fe                 │
│  │ Version: 1                                                                                    │
│  │ Digest: Ein91NF2hc3qC4XYoMUFMfin9U23xQmDAdEMSHLae7MK                                          │
│  │ Modules: todo_list                                                                            │
│  └──                                                                                             │
╰──────────────────────────────────────────────────────────────────────────────────────────────────╯
```

### 残高の変更

この最後のセクションには、SUIコインの変更が含まれています。私たちの場合、約0.015 SUIを _消費_ しており、これはMISTでは10,500,000になります。出力の _amount_ フィールドで確認できます。

```plaintext
╭───────────────────────────────────────────────────────────────────────────────────────────────────╮
│ Balance Changes                                                                                   │
├───────────────────────────────────────────────────────────────────────────────────────────────────┤
│  ┌──                                                                                              │
│  │ Owner: Account Address ( 0x091ef55506ad814920adcef32045f9078f2f6e9a72f4cf253a1e6274157380a1 )  │
│  │ CoinType: 0x2::sui::SUI                                                                        │
│  │ Amount: -10426280                                                                              │
│  └──                                                                                              │
╰───────────────────────────────────────────────────────────────────────────────────────────────────╯
```

### 代替出力

公開時に `--json` フラグを指定して、JSON形式で出力を取得することも可能です。これは、出力をプログラム的に解析したり、後で使用するために保存したりしたい場合に便利です。

```bash
$ sui client publish --gas-budget 100000000 --json
```

### 結果の使用

パッケージがオンチェーンに公開された後、パッケージと相互作用できます。これを行うには、パッケージのアドレス（オブジェクトID）を見つける必要があります。これは、`Object Changes` 出力の `Published Objects` セクションにあります。アドレスは各パッケージに固有なので、出力からコピーする必要があります。

この例では、アドレスは次のとおりです：

```plaintext
0x468daa33dfcb3e17162bbc8928f6ec73744bb08d838d1b6eb94eac99269b29fe
```

アドレスが分かったので、パッケージと相互作用できます。次のセクションでは、トランザクションを送信してパッケージと相互作用する方法を説明します。

## トランザクションの送信

`todo_list` パッケージとの相互作用をデモンストレーションするために、新しいリストを作成してアイテムを追加するトランザクションを送信します。トランザクションは `sui client ptb` コマンド経由で送信され、[トランザクションブロック](./../concepts/what-is-a-transaction)を最大限に活用できます。コマンドは大きくて複雑に見えるかもしれませんが、ステップバイステップで進めます。

### 変数を準備する

コマンドを構築する前に、トランザクションで使用する値を保存しましょう。`0x4....` を公開したパッケージのアドレスに置き換えてください。`MY_ADDRESS` 変数は、CLI出力からあなたのアドレスに自動的に設定されます。

```bash
$ export PACKAGE_ID=0x468daa33dfcb3e17162bbc8928f6ec73744bb08d838d1b6eb94eac99269b29fe
$ export MY_ADDRESS=$(sui client active-address)
```

### CLIでトランザクションを構築する

次に、実際のトランザクションを構築します。トランザクションは2つの部分で構成されます：`todo_list` パッケージの `new` 関数を呼び出して新しいリストを作成し、次にリストオブジェクトを私たちのアカウントに転送します。トランザクションは次のようになります：

```bash
$ sui client ptb \
--gas-budget 100000000 \
--assign sender @$MY_ADDRESS \
--move-call $PACKAGE_ID::todo_list::new \
--assign list \
--transfer-objects "[list]" sender
```

このコマンドでは、`ptb` サブコマンドを使用してトランザクションを構築しています。それに続くパラメータは、トランザクションが実行する実際のコマンドとアクションを定義します。最初の2つの呼び出しは、送信者アドレスをコマンド入力に設定し、トランザクションのgas予算を設定するユーティリティ呼び出しです。

```bash
# トランザクションのgas予算を設定
--gas-budget 100000000 \n
# 変数 "sender=@..." を登録
--assign sender @$MY_ADDRESS \n
```

次に、パッケージ内の関数への実際の呼び出しを実行します。パッケージID、モジュール名、関数名に続いて `--move-call` を使用します。この場合、`todo_list` パッケージの `new` 関数を呼び出しています。

```bash
# $PACKAGE_ID アドレス下の "todo_list" パッケージの "new" 関数を呼び出す
--move-call $PACKAGE_ID::todo_list::new
```

定義した関数は実際に値を返すので、それを保存したいと思います。`--assign` コマンドを使用して、戻り値に名前を付けます。この場合、`list` と呼んでいます。その後、オブジェクトを私たちのアカウントに転送します。

```bash
--move-call $PACKAGE_ID::todo_list::new \
# "new" 関数の結果を "list" 変数に割り当て（前のステップから）
--assign list \
# オブジェクトを送信者に転送
--transfer-objects "[list]" sender
```

コマンドが構築されたら、ターミナルで実行できます。すべてが正しければ、前のセクションで見たものと似た出力が表示されるはずです。出力には、トランザクションダイジェスト、トランザクションデータ、およびトランザクション効果が含まれます。

<details>
<summary><a>ネタバレ：完全なトランザクション出力</a></summary>

```bash
Transaction Digest: BJwYEnuuMzU4Y8cTwMoJbbQA6cLwPmwxvsRpSmvThoK8
╭──────────────────────────────────────────────────────────────────────────────────────────────────────────────╮
│ Transaction Data                                                                                             │
├──────────────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ Sender: 0x091ef55506ad814920adcef32045f9078f2f6e9a72f4cf253a1e6274157380a1                                   │
│ Gas Owner: 0x091ef55506ad814920adcef32045f9078f2f6e9a72f4cf253a1e6274157380a1                                │
│ Gas Budget: 100000000 MIST                                                                                  │
│ Gas Price: 1000 MIST                                                                                         │
│ Gas Payment:                                                                                                 │
│  ┌──                                                                                                         │
│  │ ID: 0xe5ddeb874a8d7ead328e9f2dd2ad8d25383ab40781a5f1aefa75600973b02bc4                                    │
│  │ Version: 22                                                                                               │
│  │ Digest: DiBrBMshDiD9cThpaEgpcYSF76uV4hCoE1qRyQ3rnYCB                                                      │
│  └──                                                                                                         │
│                                                                                                              │
│ Transaction Kind: Programmable                                                                               │
│ ╭──────────────────────────────────────────────────────────────────────────────────────────────────────────╮ │
│ │ Input Objects                                                                                            │ │
│ ├──────────────────────────────────────────────────────────────────────────────────────────────────────────┤ │
│ │ 0   Pure Arg: Type: address, Value: "0x091ef55506ad814920adcef32045f9078f2f6e9a72f4cf253a1e6274157380a1" │ │
│ ╰──────────────────────────────────────────────────────────────────────────────────────────────────────────╯ │
│ ╭──────────────────────────────────────────────────────────────────────────────────╮                         │
│ │ Commands                                                                         │                         │
│ ├──────────────────────────────────────────────────────────────────────────────────┤                         │
│ │ 0  MoveCall:                                                                     │                         │
│ │  ┌                                                                               │                         │
│ │  │ Function:  new                                                                │                         │
│ │  │ Module:    todo_list                                                          │                         │
│ │  │ Package:   0x468daa33dfcb3e17162bbc8928f6ec73744bb08d838d1b6eb94eac99269b29fe │                         │
│ │  └                                                                               │                         │
│ │                                                                                  │                         │
│ │ 1  TransferObjects:                                                              │                         │
│ │  ┌                                                                               │                         │
│ │  │ Arguments:                                                                    │                         │
│ │  │   Result 0                                                                    │                         │
│ │  │ Address: Input  0                                                             │                         │
│ │  └                                                                               │                         │
│ ╰──────────────────────────────────────────────────────────────────────────────────╯                         │
│                                                                                                              │
│ Signatures:                                                                                                  │
│    C5Lie4dtP5d3OkKzFBa+xM0BiNoB/A4ItthDCRTRBUrEE+jXeNs7mP4AuGwi3nzfTskh29+R1j1Kba4Wdy3QDA==                  │
│                                                                                                              │
╰──────────────────────────────────────────────────────────────────────────────────────────────────────────────╯
╭───────────────────────────────────────────────────────────────────────────────────────────────────╮
│ Transaction Effects                                                                               │
├───────────────────────────────────────────────────────────────────────────────────────────────────┤
│ Digest: BJwYEnuuMzU4Y8cTwMoJbbQA6cLwPmwxvsRpSmvThoK8                                              │
│ Status: Success                                                                                   │
│ Executed Epoch: 1213                                                                              │
│                                                                                                   │
│ Created Objects:                                                                                  │
│  ┌──                                                                                              │
│  │ ID: 0x74973c4ea2e78dc409f60481e23761cee68a48156df93a93fbcceb77d1cacdf6                         │
│  │ Owner: Account Address ( 0x091ef55506ad814920adcef32045f9078f2f6e9a72f4cf253a1e6274157380a1 )  │
│  │ Version: 23                                                                                    │
│  │ Digest: DuHTozDHMsuA7cFnWRQ1Gb8FQghAEBaj3inasJxqYq1c                                           │
│  └──                                                                                              │
│ Mutated Objects:                                                                                  │
│  ┌──                                                                                              │
│  │ ID: 0xe5ddeb874a8d7ead328e9f2dd2ad8d25383ab40781a5f1aefa75600973b02bc4                         │
│  │ Owner: Account Address ( 0x091ef55506ad814920adcef32045f9078f2f6e9a72f4cf253a1e6274157380a1 )  │
│  │ Version: 23                                                                                    │
│  │ Digest: 82fwKarGuDhtomr5oS6ZGNvZNw9QVXLSbPdQu6jQgNV7                                           │
│  └──                                                                                              │
│ Gas Object:                                                                                       │
│  ┌──                                                                                              │
│  │ ID: 0xe5ddeb874a8d7ead328e9f2dd2ad8d25383ab40781a5f1aefa75600973b02bc4                         │
│  │ Owner: Account Address ( 0x091ef55506ad814920adcef32045f9078f2f6e9a72f4cf253a1e6274157380a1 )  │
│  │ Version: 23                                                                                    │
│  │ Digest: 82fwKarGuDhtomr5oS6ZGNvZNw9QVXLSbPdQu6jQgNV7                                           │
│  └──                                                                                              │
│ Gas Cost Summary:                                                                                 │
│    Storage Cost: 2318000 MIST                                                                     │
│    Computation Cost: 1000000 MIST                                                                 │
│    Storage Rebate: 978120 MIST                                                                    │
│    Non-refundable Storage Fee: 9880 MIST                                                          │
│                                                                                                   │
│ Transaction Dependencies:                                                                         │
│    FSz2fYXmKqTf77mFXNq5JK7cKY8agWja7V5yDKEgL8c3                                                   │
│    GgMZKTt482DYApbAZkPDtdssGHZLbxgjm2uMXhzJax8Q                                                   │
╰───────────────────────────────────────────────────────────────────────────────────────────────────╯
╭─────────────────────────────╮
│ No transaction block events │
╰─────────────────────────────╯

╭───────────────────────────────────────────────────────────────────────────────────────────────────────╮
│ Object Changes                                                                                        │
├───────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ Created Objects:                                                                                      │
│  ┌──                                                                                                  │
│  │ ObjectID: 0x74973c4ea2e78dc409f60481e23761cee68a48156df93a93fbcceb77d1cacdf6                       │
│  │ Sender: 0x091ef55506ad814920adcef32045f9078f2f6e9a72f4cf253a1e6274157380a1                         │
│  │ Owner: Account Address ( 0x091ef55506ad814920adcef32045f9078f2f6e9a72f4cf253a1e6274157380a1 )      │
│  │ ObjectType: 0x468daa33dfcb3e17162bbc8928f6ec73744bb08d838d1b6eb94eac99269b29fe::todo_list::TodoList  │
│  │ Version: 23                                                                                        │
│  │ Digest: DuHTozDHMsuA7cFnWRQ1Gb8FQghAEBaj3inasJxqYq1c                                               │
│  └──                                                                                                  │
│ Mutated Objects:                                                                                      │
│  ┌──                                                                                                  │
│  │ ObjectID: 0xe5ddeb874a8d7ead328e9f2dd2ad8d25383ab40781a5f1aefa75600973b02bc4                       │
│  │ Sender: 0x091ef55506ad814920adcef32045f9078f2f6e9a72f4cf253a1e6274157380a1                         │
│  │ Owner: Account Address ( 0x091ef55506ad814920adcef32045f9078f2f6e9a72f4cf253a1e6274157380a1 )      │
│  │ ObjectType: 0x2::coin::Coin<0x2::sui::SUI>                                                         │
│  │ Version: 23                                                                                        │
│  │ Digest: 82fwKarGuDhtomr5oS6ZGNvZNw9QVXLSbPdQu6jQgNV7                                               │
│  └──                                                                                                  │
╰───────────────────────────────────────────────────────────────────────────────────────────────────────╯
╭───────────────────────────────────────────────────────────────────────────────────────────────────╮
│ Balance Changes                                                                                   │
├───────────────────────────────────────────────────────────────────────────────────────────────────┤
│  ┌──                                                                                              │
│  │ Owner: Account Address ( 0x091ef55506ad814920adcef32045f9078f2f6e9a72f4cf253a1e6274157380a1 )  │
│  │ CoinType: 0x2::sui::SUI                                                                        │
│  │ Amount: -2339880                                                                               │
│  └──                                                                                              │
╰───────────────────────────────────────────────────────────────────────────────────────────────────╯
```

</details>

私たちが注目したいセクションは "Object Changes" です。より具体的には、その "Created Objects" 部分です。作成した `TodoList` のオブジェクトID、タイプ、バージョンが含まれています。このオブジェクトIDを使用してリストと相互作用します。

```bash
╭───────────────────────────────────────────────────────────────────────────────────────────────────────╮
│ Object Changes                                                                                        │
├───────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ Created Objects:                                                                                      │
│  ┌──                                                                                                  │
│  │ ObjectID: 0x20e0bede16de8a728ab25e228816b9059b45ebea49c8ad384e044580b2d3e553                       │
│  │ Sender: 0x091ef55506ad814920adcef32045f9078f2f6e9a72f4cf253a1e6274157380a1                         │
│  │ Owner: Account Address ( 0x091ef55506ad814920adcef32045f9078f2f6e9a72f4cf253a1e6274157380a1 )      │
│  │ ObjectType: 0x468daa33dfcb3e17162bbc8928f6ec73744bb08d838d1b6eb94eac99269b29fe::todo_list::TodoList  │
│  │ Version: 22                                                                                        │
│  │ Digest: HyWdUpjuhjLY38dLpg6KPHQ3bt4BqQAbdF5gB8HQdEqG                                               │
│  └──                                                                                                  │
│ Mutated Objects:                                                                                      │
│  ┌──                                                                                                  │
│  │ ObjectID: 0xe5ddeb874a8d7ead328e9f2dd2ad8d25383ab40781a5f1aefa75600973b02bc4                       │
│  │ Sender: 0x091ef55506ad814920adcef32045f9078f2f6e9a72f4cf253a1e6274157380a1                         │
│  │ Owner: Account Address ( 0x091ef55506ad814920adcef32045f9078f2f6e9a72f4cf253a1e6274157380a1 )      │
│  │ ObjectType: 0x2::coin::Coin<0x2::sui::SUI>                                                         │
│  │ Version: 22                                                                                        │
│  │ Digest: DiBrBMshDiD9cThpaEgpcYSF76uV4hCoE1qRyQ3rnYCB                                               │
│  └──                                                                                                  │
╰───────────────────────────────────────────────────────────────────────────────────────────────────────╯
```

この例では、オブジェクトIDは
`0x20e0bede16de8a728ab25e228816b9059b45ebea49c8ad384e044580b2d3e553` です。そして、所有者はあなたのアカウントアドレスであるべきです。これは、トランザクションの最後のコマンドでオブジェクトを送信者に転送することで達成しました。

リストの作成に成功したことをテストする別の方法は、アカウントオブジェクトを確認することです。

```bash
$ sui client objects
```

次のようなオブジェクトがあるはずです：

```plaintext
╭  ...                                                                                  ╮
│ ╭────────────┬──────────────────────────────────────────────────────────────────────╮ │
│ │ objectId   │  0x20e0bede16de8a728ab25e228816b9059b45ebea49c8ad384e044580b2d3e553  │ │
│ │ version    │  22                                                                  │ │
│ │ digest     │  /DUEiCLkaNSgzpZSq2vSV0auQQEQhyH9occq9grMBZM=                        │ │
│ │ objectType │  0x468d..29fe::todo_list::TodoList                                   │ │
│ ╰────────────┴──────────────────────────────────────────────────────────────────────╯ │
|  ...                                                                                  |
```

### オブジェクトを関数に渡す

前のステップで作成したTodoListは、所有者として相互作用できるオブジェクトです。このオブジェクトに対して、`todo_list` モジュールで定義された関数を呼び出すことができます。これをデモンストレーションするために、リストにアイテムを追加します。まず1つのアイテムだけを追加し、2番目のトランザクションでは3つ追加して、1つを削除します。

[前のステップ](#prepare-the-variables)で変数が設定されていることを再度確認し、リストオブジェクト用の変数をもう1つ追加します。

```bash
$ export LIST_ID=0x20e0bede16de8a728ab25e228816b9059b45ebea49c8ad384e044580b2d3e553
```

今度は、リストにアイテムを追加するトランザクションを構築できます。コマンドは次のようになります：

```bash
$ sui client ptb \
--gas-budget 100000000 \
--move-call $PACKAGE_ID::todo_list::add @$LIST_ID "'Finish the Hello, Sui chapter'"
```

このコマンドでは、`todo_list` パッケージの `add` 関数を呼び出しています。この関数は2つの引数を取ります：リストオブジェクトと追加するアイテムです。アイテムは文字列なので、シングルクォートで囲む必要があります。コマンドはリストにアイテムを追加します。

すべてが正しければ、前のセクションで見たものと似た出力が表示されるはずです。
今度は、リストオブジェクトを確認して、アイテムが追加されたかどうか確認できます。

```bash
$ sui client object $LIST_ID
```

出力には、追加したアイテムが含まれているはずです。

```plaintext
╭───────────────┬───────────────────────────────────────────────────────────────────────────────────────────────────────────────────╮
│ objectId      │  0x20e0bede16de8a728ab25e228816b9059b45ebea49c8ad384e044580b2d3e553                                               │
│ version       │  24                                                                                                               │
│ digest        │  FGcXH8MGpMs5BdTnC62CQ3VLAwwexYg2id5DKU7Jr9aQ                                                                     │
│ objType       │  0x468daa33dfcb3e17162bbc8928f6ec73744bb08d838d1b6eb94eac99269b29fe::todo_list::TodoList                          │
│ owner         │ ╭──────────────┬──────────────────────────────────────────────────────────────────────╮                           │
│               │ │ AddressOwner │  0x091ef55506ad814920adcef32045f9078f2f6e9a72f4cf253a1e6274157380a1  │                           │
│               │ ╰──────────────┴──────────────────────────────────────────────────────────────────────╯                           │
│ prevTx        │  EJVK6FEHtfTdCuGkNsU1HcrmUBEN6H6jshfcptnw8Yt1                                                                     │
│ storageRebate │  1558000                                                                                                          │
│ content       │ ╭───────────────────┬───────────────────────────────────────────────────────────────────────────────────────────╮ │
│               │ │ dataType          │  moveObject                                                                               │ │
│               │ │ type              │  0x468daa33dfcb3e17162bbc8928f6ec73744bb08d838d1b6eb94eac99269b29fe::todo_list::TodoList  │ │
│               │ │ hasPublicTransfer │  true                                                                                     │ │
│               │ │ fields            │ ╭───────┬───────────────────────────────────────────────────────────────────────────────╮ │ │
│               │ │                   │ │ id    │ ╭────┬──────────────────────────────────────────────────────────────────────╮ │ │ │
│               │ │                   │ │       │ │ id │  0x20e0bede16de8a728ab25e228816b9059b45ebea49c8ad384e044580b2d3e553  │ │ │ │
│               │ │                   │ │       │ ╰────┴──────────────────────────────────────────────────────────────────────╯ │ │ │
│               │ │                   │ │ items │ ╭─────────────────────────────────╮                                           │ │ │
│               │ │                   │ │       │ │  finish the Hello, Sui chapter  │                                           │ │ │
│               │ │                   │ │       │ ╰─────────────────────────────────╯                                           │ │ │
│               │ │                   │ ╰───────┴───────────────────────────────────────────────────────────────────────────────╯ │ │
│               │ ╰───────────────────┴───────────────────────────────────────────────────────────────────────────────────────────╯ │
╰───────────────┴───────────────────────────────────────────────────────────────────────────────────────────────────────────────────╯
```

オブジェクトのJSON表現は、コマンドに `--json` フラグを追加することで取得できます。

```bash
$ sui client object $LIST_ID --json
```

```json
{
  "objectId": "0x20e0bede16de8a728ab25e228816b9059b45ebea49c8ad384e044580b2d3e553",
  "version": "24",
  "digest": "FGcXH8MGpMs5BdTnC62CQ3VLAwwexYg2id5DKU7Jr9aQ",
  "type": "0x468daa33dfcb3e17162bbc8928f6ec73744bb08d838d1b6eb94eac99269b29fe::todo_list::TodoList",
  "owner": {
    "AddressOwner": "0x091ef55506ad814920adcef32045f9078f2f6e9a72f4cf253a1e6274157380a1"
  },
  "previousTransaction": "EJVK6FEHtfTdCuGkNsU1HcrmUBEN6H6jshfcptnw8Yt1",
  "storageRebate": "1558000",
  "content": {
    "dataType": "moveObject",
    "type": "0x468daa33dfcb3e17162bbc8928f6ec73744bb08d838d1b6eb94eac99269b29fe::todo_list::TodoList",
    "hasPublicTransfer": true,
    "fields": {
      "id": {
        "id": "0x20e0bede16de8a728ab25e228816b9059b45ebea49c8ad384e044580b2d3e553"
      },
      "items": ["Finish the Hello, Sui chapter"]
    }
  }
}
```

### コマンドの連鎖

単一のトランザクションで複数のコマンドを連鎖することができます。これはトランザクションブロックの威力を示しています！
同じリストオブジェクトを使用して、3つのアイテムを追加し、1つを削除します。コマンドは次のようになります：

```bash
$ sui client ptb \
--gas-budget 100000000 \
--move-call $PACKAGE_ID::todo_list::add @$LIST_ID "'Finish Concepts chapter'" \
--move-call $PACKAGE_ID::todo_list::add @$LIST_ID "'Read the Move Basics chapter'" \
--move-call $PACKAGE_ID::todo_list::add @$LIST_ID "'Learn about Object Model'" \
--move-call $PACKAGE_ID::todo_list::remove @$LIST_ID 0
```

前のコマンドが成功していれば、このコマンドも何ら変わりはないはずです。リストオブジェクトを確認して、アイテムが追加され、削除されたかどうか確認できます。JSON表現の方が少し読みやすいです！

```bash
sui client object $LIST_ID --json
```

```json
{
  "objectId": "0x20e0bede16de8a728ab25e228816b9059b45ebea49c8ad384e044580b2d3e553",
  "version": "25",
  "digest": "EDTXDsteqPGAGu4zFAj5bbQGTkucWk4hhuUquk39enGA",
  "type": "0x468daa33dfcb3e17162bbc8928f6ec73744bb08d838d1b6eb94eac99269b29fe::todo_list::TodoList",
  "owner": {
    "AddressOwner": "0x091ef55506ad814920adcef32045f9078f2f6e9a72f4cf253a1e6274157380a1"
  },
  "previousTransaction": "7SXLGBSh31jv8G7okQ9mEgnw5MnTfvzzHEHpWf3Sa9gY",
  "storageRebate": "1922800",
  "content": {
    "dataType": "moveObject",
    "type": "0x468daa33dfcb3e17162bbc8928f6ec73744bb08d838d1b6eb94eac99269b29fe::todo_list::TodoList",
    "hasPublicTransfer": true,
    "fields": {
      "id": {
        "id": "0x20e0bede16de8a728ab25e228816b9059b45ebea49c8ad384e044580b2d3e553"
      },
      "items": [
        "Finish Concepts chapter",
        "Read the Move Basics chapter",
        "Learn about Object Model"
      ]
    }
  }
}
```

コマンドは同じパッケージ内である必要はなく、同じオブジェクトで動作する必要もありません。単一のトランザクションブロック内で、複数のパッケージやオブジェクトと相互作用できます。これは、オンチェーンで複雑な相互作用を構築できる強力な機能です！

## 結論

このガイドでは、Moveブロックチェーンにパッケージを公開し、Sui CLIを使用してそれと相互作用する方法を説明しました。新しいリストオブジェクトを作成し、それにアイテムを追加し、削除する方法をデモンストレーションしました。また、単一のトランザクションブロックで複数のコマンドを連鎖する方法も説明しました。このガイドは、Suiブロックチェーン上で独自のアプリケーションを構築するための良い出発点を提供するはずです！
