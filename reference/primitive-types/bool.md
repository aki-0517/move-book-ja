---
title: 'Bool | リファレンス'
description: ''
---

# Bool

`bool`はMoveのブール値`true`と`false`のプリミティブ型です。

## リテラル

`bool`のリテラルは`true`または`false`のどちらかです。

## 操作

### 論理演算

`bool`は3つの論理演算をサポートします：

| 構文                    | 説明                  | 同等の式                                               |
| ------------------------- | ---------------------------- | ------------------------------------------------------------------- |
| `&&`                      | ショートサーキット論理AND | `p && q`は`if (p) q else false`と同等                     |
| <code>&vert;&vert;</code> | ショートサーキット論理OR  | <code>p &vert;&vert; q</code>は`if (p) true else q`と同等 |
| `!`                       | 論理否定             | `!p`は`if (p) false else true`と同等                      |

### 制御フロー

`bool`値はMoveのいくつかの制御フロー構造で使用されます：

- [`if (bool) { ... }`](./../control-flow/conditionals)
- [`while (bool) { .. }`](./../control-flow/loops)
- [`assert!(bool, u64)`](./../abort-and-assert)

## 所有権

言語に組み込まれた他のスカラー値と同様に、ブール値は暗黙的にコピー可能であり、[`copy`](.././variables#move-and-copy)などの明示的な命令なしにコピーできることを意味します。
