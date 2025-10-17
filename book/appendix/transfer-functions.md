# 付録C: 転送関数

## 転送関数の比較

| 関数                      | パブリック関数              | 終了状態      | 権限                       |
| ------------------------- | --------------------------- | ------------- | -------------------------- |
| [`transfer`][transfer]    | `public_transfer`           | アドレス所有  | 完全                       |
| [`share_object`][share]   | `public_share_object`       | 共有          | Ref, Mut Ref, Delete       |
| [`freeze_object`][freeze] | `public_freeze_object`      | 凍結          | Ref                        |
| [`party_transfer`][party] | `public_party_transfer`     | パーティー    | [パーティーテーブル参照](#party) |

## 状態の比較

| 状態          | 説明                                                       |
| ------------- | ---------------------------------------------------------- |
| アドレス所有  | オブジェクトはアドレス（またはオブジェクト）によって完全にアクセス可能 |
| 共有          | オブジェクトは誰でも参照・削除可能                         |
| 凍結          | オブジェクトは不変参照を通じてアクセス可能                 |
| パーティー    | パーティー設定に依存（[パーティーテーブル参照](#party)）   |

## パーティー

| 関数           | 説明                                       |
| -------------- | ------------------------------------------ |
| `single_owner` | オブジェクトはアドレス所有と同じ権限を持つ |

[transfer]: https://docs.sui.io/references/framework/sui_sui/transfer#sui_transfer_transfer
[share]: https://docs.sui.io/references/framework/sui_sui/transfer#sui_transfer_share_object
[freeze]: https://docs.sui.io/references/framework/sui_sui/transfer#sui_transfer_freeze_object
[party]: https://docs.sui.io/references/framework/sui_sui/transfer#sui_transfer_party_transfer
