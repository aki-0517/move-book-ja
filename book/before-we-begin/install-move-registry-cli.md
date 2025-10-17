# MVRのインストール

[Move Registry (MVR)](https://moveregistry.com)は、Move用のパッケージマネージャーです。誰でもMoveで書かれた新しいアプリケーションで公開されたパッケージを公開し、使用できます。ローカルバイナリは、レジストリでパッケージを検索し、Sui CLIビルドプロセスの一部としてそれらをインストールすることを可能にします。

## suiupを使用したインストール

MVRをインストールする最良の方法は、[`suiup`](https://github.com/MystenLabs/suiup)を使用することです。Suiupは、バイナリの異なるバージョンを更新・管理する簡単な方法を提供します。

`suiup`のインストール手順は[リポジトリのREADME](https://github.com/MystenLabs/suiup)で見つけることができます。

Move Registry CLIをインストールするには、以下のコマンドを実行してください：

```bash
suiup install mvr
```

インストール後、Move Registryは`mvr`として利用可能になります。

## バイナリのダウンロード

[リリースページ](https://github.com/MystenLabs/mvr/releases)から最新のMVRバイナリをダウンロードできます。バイナリはmacOS、Linux、Windowsで利用可能です。[Sui](./install-sui.md)とは異なり、MVRバイナリは環境間で変更されず、`testnet`と`mainnet`の両方をサポートします。

## Cargoを使用したインストール

Cargo（Rustが必要）を使用してMVRをローカルにインストール・ビルドできます

```bash
cargo install --locked --git https://github.com/mystenlabs/mvr --branch release mvr
```

## トラブルシューティング

インストールプロセスのトラブルシューティングについては、[MVRのインストール](https://docs.suins.io/move-registry/tooling/mvr-cli#installation)ガイドを参照してください。