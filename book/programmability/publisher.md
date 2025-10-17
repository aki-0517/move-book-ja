# パブリッシャー権限

アプリケーションの設計と開発では、パブリッシャー権限を証明する必要がよくあります。
これは、デジタルアセットの文脈で特に重要です。パブリッシャーはアセットの特定の機能を
有効または無効にできるからです。Publisher Objectは[Sui Framework](./sui-framework)で
定義されたオブジェクトで、パブリッシャーが_型に対する権限_を証明することを可能にします。

## 定義

Publisherオブジェクトは、Sui Frameworkの`sui::package`モジュールで定義されています。
これは非常にシンプルな非ジェネリックオブジェクトで、モジュールごとに一度（パッケージごとに
複数回）初期化でき、型に対するパブリッシャーの権限を証明するために使用されます。
Publisherオブジェクトを要求するには、パブリッシャーは`package::claim`関数に
[One Time Witness](./one-time-witness)を提示する必要があります。

```move
module sui::package;

public struct Publisher has key, store {
    id: UID,
    package: String,
    module_name: String,
}
```

> One Time Witnessに慣れていない場合は、[こちら](./one-time-witness)で詳しく読むことができます。

以下は、モジュールで`Publisher`オブジェクトを要求する簡単な例です：

```move file=packages/samples/sources/programmability/publisher.move anchor=publisher

```

## 使用法

Publisherオブジェクトには、型に対するパブリッシャーの権限を証明するために使用される
2つの関数が関連付けられています：

```move file=packages/samples/sources/programmability/publisher.move anchor=use_publisher

```

## 管理者ロールとしてのPublisher

小さなアプリケーションやシンプルなユースケースでは、Publisherオブジェクトを管理者
[ケーパビリティ](./capability)として使用できます。より広い文脈では、Publisherオブジェクトは
システム設定を制御しますが、アプリケーションの状態を管理するためにも使用できます。

```move file=packages/samples/sources/programmability/publisher.move anchor=publisher_as_admin

```

ただし、Publisherは型安全性や表現力など、[ケーパビリティ](./capability)の一部の
ネイティブプロパティが欠けています。`admin_action`のシグネチャはそれほど明示的ではなく、
他の誰でも呼び出すことができます。そして、`Publisher`オブジェクトが標準であるため、
`from_module`チェックが実行されない場合、不正アクセスのリスクがあります。
したがって、`Publisher`オブジェクトを管理者ロールとして使用する際は注意が必要です。

## Suiでの役割

Publisherは、Suiの特定の機能に必要です。[Object Display](./display)はPublisherによってのみ
作成でき、Kioskシステムの重要なコンポーネントであるTransferPolicyも、型の所有権を証明するために
Publisherオブジェクトを必要とします。

## 次のステップ

次の章では、Publisherオブジェクトを必要とする最初の機能 - Object Display - について説明します。
これは、クライアント用にオブジェクトを記述し、メタデータを標準化する方法です。
ユーザーフレンドリーなアプリケーションには必須です。
