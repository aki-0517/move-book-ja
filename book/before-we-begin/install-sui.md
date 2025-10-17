# Suiのインストール

Moveはコンパイル言語のため、Moveプログラムを書いて実行するためにはコンパイラをインストールする必要があります。コンパイラはSuiバイナリに含まれており、以下の方法のいずれかを使用してインストールまたはダウンロードできます。

## suiupを使ったインストール

Suiをインストールする最良の方法は[`suiup`](https://github.com/MystenLabs/suiup)を使用することです。異なる環境（例：`testnet`と`mainnet`）用のバイナリをインストールし、異なるバージョンのバイナリを管理するシンプルな方法を提供します。

`suiup`のインストール手順は[リポジトリのREADME](https://github.com/MystenLabs/suiup)にあります。

Suiをインストールするには、以下のコマンドを実行します：

```bash
suiup install sui
```

## バイナリのダウンロード

最新のSuiバイナリは[リリースページ](https://github.com/MystenLabs/sui/releases)からダウンロードできます。バイナリはmacOS、Linux、Windows用が利用可能です。教育目的や開発には、`mainnet`版の使用をお勧めします。

## Homebrewを使ったインストール（MacOS）

[Homebrew](https://brew.sh/)パッケージマネージャーを使ってSuiをインストールできます。

```bash
brew install sui
```

## Chocolateyを使ったインストール（Windows）

Windows用の[Chocolatey](https://chocolatey.org/install)パッケージマネージャーを使ってSuiをインストールできます。

```bash
choco install sui
```

## Cargoを使ったビルド（MacOS、Linux）

Cargoパッケージマネージャーを使ってSuiをローカルでインストール・ビルドできます（Rustが必要）

```bash
cargo install --git https://github.com/MystenLabs/sui.git sui --branch mainnet
```

これらのいずれかをターゲットにしている場合は、ここでブランチターゲットを`testnet`または`devnet`に変更してください。

以下のコマンドでシステムに最新のRustバージョンがあることを確認してください。

```bash
rustup update stable
```

## トラブルシューティング

インストールプロセスのトラブルシューティングについては、[Install Sui](https://docs.sui.io/guides/developer/getting-started/sui-install) ガイドを参照してください。
