# トランザクション（Transaction）

トランザクションはブロックチェーンの世界における基本的な概念です。これはブロックチェーンと対話する方法です。トランザクションはブロックチェーンの状態を変更するために使用され、それを行う唯一の方法です。Moveでは、トランザクションはパッケージ内の関数を呼び出し、新しいパッケージをデプロイし、既存のものをアップグレードするために使用されます。

<!--

- how user interacts with a program
    - mention public functions
    - give a concept of an entry / public function without getting into details
    - mention that functions are called in transactions
    - mention that transactions are sent by accounts
    - every transaction specifies object it operates on

 -->

## トランザクション構造

> 全てのトランザクションは、操作するオブジェクトを明示的に指定します！

トランザクションは以下で構成されます：

- 送信者 - トランザクションに_署名_する[アカウント](./what-is-an-account)
- コマンドのリスト（またはチェーン） - 実行される操作
- コマンド入力 - コマンドの引数：数値や文字列のような単純な値の`pure`、またはトランザクションがアクセスするオブジェクトの`object`
- ガスオブジェクト - トランザクションの支払いに使用される`Coin`オブジェクト
- ガス価格と予算 - トランザクションのコスト

## 入力

トランザクション入力はトランザクションの引数で、2つのタイプに分けられます：

- Pure引数：これらは主に[プリミティブ型](../move-basics/primitive-types)にいくつかの追加機能があります。Pure引数は以下が可能です：
  - [`bool`](../move-basics/primitive-types#booleans)
  - [符号なし整数](../move-basics/primitive-types#integer-types)（`u8`、`u16`、`u32`、`u64`、`u128`、`u256`）
  - [`address`](../move-basics/address)
  - [`std::string::String`](../move-basics/string)、UTF8文字列
  - [`std::ascii::String`](../move-basics/string#ascii-strings)、ASCII文字列
  - [`vector<T>`](../move-basics/vector)、ここで`T`はpure型
  - [`std::option::Option<T>`](../move-basics/option)、ここで`T`はpure型
  - [`std::object::ID`](../storage/uid-and-id)、通常はオブジェクトを指す。[オブジェクトとは何か](../object/object-model)も参照
- オブジェクト引数：これらはトランザクションがアクセスするオブジェクトまたはオブジェクトの参照です。オブジェクト引数は、トランザクションが成功するために、共有オブジェクト、凍結オブジェクト、またはトランザクション送信者が所有するオブジェクトである必要があります。詳細については[オブジェクトモデル](../object)を参照してください。

## コマンド

Suiトランザクションは複数のコマンドで構成される場合があります。各コマンドは、単一の組み込みコマンド（パッケージの公開など）または既に公開されたパッケージ内の関数の呼び出しです。コマンドはトランザクション内にリストされた順序で実行され、前のコマンドの結果を使用してチェーンを形成できます。トランザクションは全体として成功または失敗します。

概念的に、トランザクションは次のようになります（疑似コード）：

```
Inputs:
- sender = 0xa11ce

Commands:
- payment = SplitCoins(Gas, [ 1000 ])
- item = MoveCall(0xAAA::market::purchase, [ payment ])
- TransferObjects(item, sender)
```

この例では、トランザクションは3つのコマンドで構成されます：

1. `SplitCoins` - 渡されたオブジェクト（この場合は`Gas`オブジェクト）から新しいコインを分割する組み込みコマンド
2. `MoveCall` - パッケージ`0xAAA`のモジュール`market`内の関数`purchase`を、指定された引数（`payment`オブジェクト）で呼び出すコマンド
3. `TransferObjects` - オブジェクトを受信者に転送する組み込みコマンド

<!--
> There are multiple different implementations of transaction building, for example
-->

## トランザクション効果

トランザクション効果は、トランザクションがブロックチェーンの状態に加える変更です。より具体的には、トランザクションは以下の方法で状態を変更できます：

- トランザクションの支払いにガスオブジェクトを使用
- オブジェクトの作成、更新、削除
- イベントの発行

実行されたトランザクションの結果は、異なる部分で構成されます：

- トランザクションダイジェスト - トランザクションを識別するために使用されるトランザクションのハッシュ
- トランザクションデータ - トランザクションで使用される入力、コマンド、ガスオブジェクト
- トランザクション効果 - トランザクションのステータスと「効果」。より具体的には：トランザクションのステータス、オブジェクトとその新しいバージョンの更新、使用されたガスオブジェクト、トランザクションのガスコスト、トランザクションによって発行されたイベント
- イベント - トランザクションによって発行されたカスタム[イベント](./../programmability/events)
- オブジェクトの変更 - _所有権の変更_を含むオブジェクトに加えられた変更
- 残高の変更 - トランザクションに関与するアカウントの総残高に加えられた変更
