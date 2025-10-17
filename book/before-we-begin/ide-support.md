# IDEのセットアップ

Move開発で最も人気のあるIDEは2つあります：VSCodeとIntelliJ IDEAです。どちらも構文ハイライトやエラーメッセージなどの基本機能を提供しますが、追加機能では違いがあります。どのIDEを選択しても、[Move CLI](./install-sui.md)を実行するためにターミナルを使用する必要があります。

> **IntelliJプラグインはMove 2024エディションをサポートしていないため、一部の構文がハイライトされません。**

## VSCode

- [VSCode](https://code.visualstudio.com/)はMicrosoftの無料でオープンソースのIDEです。
- [Move (拡張)](https://marketplace.visualstudio.com/items?itemName=mysten.move)は[Mysten Labs](https://mystenlabs.com)が保守するMove用の言語サーバー拡張です。
- [Move Formatter](https://marketplace.visualstudio.com/items?itemName=mysten.prettier-move) - [Mysten Labs](https://mystenlabs.com)が開発・保守するMove用のコードフォーマッターです。
- [Move Syntax](https://marketplace.visualstudio.com/items?itemName=damirka.move-syntax)は[Damir Shamanaev](https://github.com/damirka/)によるMove用のシンプルな構文ハイライト拡張です。

## IntelliJ IDEA

- [IntelliJ IDEA](https://www.jetbrains.com/idea/)はJetBrainsの商用IDEです。
- [Move Language Plugin](https://plugins.jetbrains.com/plugin/23301-sui-move-language)は[MoveFuns](https://movefuns.org/)によるIntelliJ IDEA用のMove on Sui言語拡張を提供します。

## Emacs

- [Emacs](https://www.gnu.org/software/emacs/)は無料でオープンソースのテキストエディターです。
- [move-mode](https://github.com/amnn/move-mode)は[Ashok Menon](https://github.com/amnn)によるEmacs用のMoveモードです。

## Zed

- [Zed](https://zed.dev/)は人間とAIとの高性能コラボレーション用に設計された次世代コードエディターです。
- [Move](https://github.com/Tzal3x/move-zed-extension)は[Tzal3x](https://github.com/Tzal3x)が保守するMove用の言語サーバー拡張です。

## Github Codespaces

GithubのWebベースIDEはブラウザで直接実行でき、ほぼフル機能のVSCode体験を提供します。

- [Github Codespaces](https://github.com/features/codespaces)
- [Move Syntax](https://marketplace.visualstudio.com/items?itemName=damirka.move-syntax)も拡張マーケットプレイスで利用できます。
- [Move Formatter](https://marketplace.visualstudio.com/items?itemName=mysten.prettier-move)も拡張マーケットプレイスで利用できます。

## その他（CLI）

上記のツールの一部にはCLIサポート版があります。

- [prettier-plugin-move](https://www.npmjs.com/package/@mysten/prettier-plugin-move)にはPrettier@v3プラグイン用のTypeScriptパッケージと、ターミナルで実行するためのバイナリが含まれています
