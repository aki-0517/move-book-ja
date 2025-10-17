# パターン：ホットポテト

アビリティシステムのケース - アビリティを持たない構造体 - は_ホットポテト_と呼ばれます。
格納できません（[オブジェクト](./../storage/key-ability)としても[他の構造体のフィールド](./../storage/store-ability)としても）、
[コピー](./../move-basics/copy-ability)や[破棄](./../move-basics/drop-ability)もできません。
したがって、一度構築されると、そのモジュールによって適切に[アンパック](./../move-basics/struct)されるか、
ドロップなしの未使用値によりトランザクションがアボートします。

> _コールバック_をサポートする言語に慣れている場合、ホットポテトをコールバック関数を
> 呼び出す義務として考えることができます。呼び出さないと、トランザクションがアボートします。

名前は、ボールがプレイヤー間で素早く渡される子供のゲームから来ており、音楽が止まったときに
最後にボールを持っているプレイヤーは誰もいません。そうでなければ、ゲームから外れます。
これがパターンの最良の例です - ホットポテト構造体のインスタンスは呼び出し間で渡され、
どのモジュールもそれを保持できません。

## ホットポテトの定義

ホットポテトは、アビリティを持たない任意の構造体にすることができます。例えば、以下の構造体は
ホットポテトです：

```move file=packages/samples/sources/programmability/hot-potato-pattern.move anchor=definition

```

`Request`にはアビリティがなく、格納や無視ができないため、モジュールはそれをアンパックする
関数を提供する必要があります。例えば：

```move file=packages/samples/sources/programmability/hot-potato-pattern.move anchor=new_request

```

## 使用例

以下の例では、`Promise`ホットポテトを使用して、借用された値がコンテナから取り出されたときに
コンテナに返されることを保証しています。`Promise`構造体には借用されたオブジェクトのIDと
コンテナのIDが含まれており、借用された値が他のものと交換されずに正しいコンテナに
返されることを保証しています。

```move file=packages/samples/sources/programmability/hot-potato-pattern.move anchor=container_borrow

```

## アプリケーション

以下に、ホットポテトパターンの一般的なユースケースをリストアップします。

### 借用

[上記の例](#example-usage)で示されているように、ホットポテトは借用された値が正しいコンテナに
返されることを保証する借用に非常に効果的です。例は`Option`内に格納された値に焦点を当てていますが、
同じパターンは他の任意のストレージ型、例えば[動的フィールド](./dynamic-fields)に適用できます。

### フラッシュローン

ホットポテトパターンの典型的な例はフラッシュローンです。フラッシュローンは、同じトランザクションで
借用され返済されるローンです。借用された資金は何らかの操作を実行するために使用され、
返済された資金は貸し手に返されます。ホットポテトパターンは、借用された資金が貸し手に
返されることを保証します。

このパターンの使用例は以下のようになります：

```move
// 貸し手から資金を借用する。
let (asset_a, potato) = lender.borrow(amount);

// 借用された資金で何らかの操作を実行する。
let asset_b = dex.trade(loan);
let proceeds = another_contract::do_something(asset_b);

// 手数料を保持し、残りを貸し手に返す。
let pay_back = proceeds.split(amount, ctx);
lender.repay(pay_back, potato);
transfer::public_transfer(proceeds, ctx.sender());
```

### 可変パス実行

ホットポテトパターンは、実行パスに変化を導入するために使用できます。例えば、
「ボーナスポイント」またはUSDで`Phone`を購入できるモジュールがある場合、
ホットポテトを使用して購入と支払いを分離できます。このアプローチは、一部の店舗の
仕組みと非常によく似ています - 棚から商品を取り、その後レジで支払います。

```move file=packages/samples/sources/programmability/hot-potato-pattern.move anchor=phone_shop

```

この分離技術により、購入ロジックと支払いロジックを分離でき、コードをよりモジュール化し、
メンテナンスしやすくします。`Ticket`は独自のモジュールに分割でき、支払いのための
基本的なインターフェースを提供し、店舗の実装は支払いロジックを変更することなく
他の商品をサポートするように拡張できます。

### 合成パターン

ホットポテトは、異なるモジュールを合成的方法でリンクするために使用できます。
そのモジュールは、ホットポテトと相互作用する方法を定義できます。例えば、
型シグネチャでスタンプするか、そこから何らかの情報を抽出します。このようにして、
ホットポテトは異なるモジュール間、さらには同じトランザクション内の異なるパッケージ間で
渡すことができます。

<!-- TODO: add [Request Pattern](./request-pattern)

The most important compositional pattern is the Request Pattern, which we will cover in the next
section. -->

### Sui Frameworkでの使用

このパターンは、Sui Frameworkで様々な形で使用されています。以下にいくつかの例を示します：

- [sui::borrow][borrow-framework] - 借用された値が正しいコンテナに返されることを保証するために
  ホットポテトを使用します。
- [sui::transfer_policy][transfer-policy-framework] - `TransferRequest`を定義します - すべての条件が
  満たされた場合にのみ消費できるホットポテトです。
- [sui::token][token-framework] - クローズドループトークンシステムでは、`ActionRequest`が
  実行されたアクションに関する情報を運び、`TransferRequest`と同様に承認を収集します。

[borrow-framework]: https://docs.sui.io/references/framework/sui-framework/borrow
[transfer-policy-framework]: https://docs.sui.io/references/framework/sui-framework/transfer_policy
[token-framework]: https://docs.sui.io/references/framework/sui-framework/token

## まとめ

- ホットポテトはアビリティを持たない構造体で、それを作成し破棄する方法と一緒に来る必要があります。
- ホットポテトは、コールバックと同様に、トランザクションが終了する前に何らかのアクションが
  実行されることを保証するために使用されます。
- ホットポテトの最も一般的なユースケースは、借用、フラッシュローン、可変パス実行、
  合成パターンです。
