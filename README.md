# The Move Book (日本語)

これは[The Move Book](https://move-book.com)と[Move Language Reference](https://move-book.com/reference)の日本語版リポジトリです。

## 構造

- 2つの本が`book`と`reference`ディレクトリに配置されています。`book`ディレクトリにはメインの本が含まれ、`reference`ディレクトリにはリファレンス本が含まれています。
- `packages`ディレクトリには、両方の本で使用されるコードサンプルが含まれています。
- `site`ディレクトリには[docusaurus](docusaurus.io)の設定とそのためのカスタムプラグインが含まれています。

## 本をローカルで実行する

### 前提条件

- NodeJS
- `pnpm`（インストール方法：`npm i -g pnpm`）

### ローカルサーバー

> すべてのコマンドはルートから実行できます。

```bash
pnpm start
```

_本は`http://localhost:3000`で利用可能になります。_

### プロダクションビルドのテスト

```bash
pnpm build
pnpm serve
```

## アーカイブ

本の旧バージョンのアーカイブについては、`archive`ブランチを参照してください。