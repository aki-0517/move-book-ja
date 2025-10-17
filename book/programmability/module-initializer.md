# モジュールイニシャライザー

多くのアプリケーションで一般的なユースケースは、パッケージが公開されたときに特定のコードを
一度だけ実行することです。公開時にメインのStoreオブジェクトを作成する必要がある
シンプルなストアモジュールを想像してください。Suiでは、これはモジュール内で`init`関数を
定義することで実現されます。この関数は、モジュールが公開されたときに自動的に呼び出されます。

> すべてのモジュールの`init`関数は、公開プロセス中に呼び出されます。現在、この動作は
> 公開コマンドに限定されており、パッケージアップグレードには拡張されません。
>
> <!-- [package upgrades]() -->

```move file=packages/samples/sources/programmability/module-initializer.move anchor=main

```

同じパッケージ内で、別のモジュールが独自の`init`関数を持ち、異なるロジックをカプセル化できます。

```move file=packages/samples/sources/programmability/module-initializer-2.move anchor=other

```

## `init`の機能

関数は、モジュールに存在し、ルールに従う場合、公開時に呼び出されます：

- 関数は`init`という名前で、プライベートで、戻り値を持たない必要があります。
- 1つまたは2つの引数を取ります：[One Time Witness](./one-time-witness)（オプション）と
  [TxContext](./transaction-context)。`TxContext`は常に最後の引数です。

```move
fun init(ctx: &mut TxContext) { /* ... */}
fun init(otw: OTW, ctx: &mut TxContext) { /* ... */ }
```

TxContextは不変参照として渡すこともできます：`&TxContext`。ただし、実際には、`init`関数は
オンチェーン状態にアクセスできず、新しいオブジェクトを作成するにはコンテキストへの可変参照が
必要であるため、常に`&mut TxContext`である必要があります。

```move
fun init(ctx: &TxContext) { /* ... */}
fun init(otw: OTW, ctx: &TxContext) { /* ... */ }
```

## 信頼とセキュリティ

`init`関数を使用して機密オブジェクトを一度作成できますが、同じオブジェクト（例：最初の例の
`StoreOwnerCap`）は別の関数で作成できることを知ることが重要です。特に、アップグレード中に
モジュールに新しい関数を追加できることを考慮するとです。したがって、`init`関数は
モジュールの初期状態を設定するのに適した場所ですが、それ自体はセキュリティ対策ではありません。

オブジェクトが一度だけ作成されることを保証する方法があります。例えば、
[One Time Witness](./one-time-witness)です。また、モジュールのアップグレードを制限または
無効にする方法もあります。これについては、Package Upgrades章で説明します。

## 次のステップ

定義からわかるように、`init`関数は、モジュールが公開されたときに一度だけ呼び出されることが
保証されています。したがって、モジュールのオブジェクトを初期化し、環境と設定をセットアップする
コードを配置するのに適した場所です。

例えば、特定のアクションに必要な[Capability](./capability)がある場合、
`init`関数で作成する必要があります。次の章では、`Capability`パターンについて
より詳しく説明します。
