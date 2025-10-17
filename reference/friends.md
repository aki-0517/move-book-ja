---
title: 'Friends | リファレンス'
description: ''
---

# 非推奨: Friends

注意: この機能は[`public(package)`](./functions#visibility)に置き換えられました。

`friend`構文は、現在のモジュールによって信頼されるモジュールを宣言するために使用されていました。信頼されたモジュールは、`public(friend)`可視性を持つ現在のモジュールで定義された任意の関数を呼び出すことができます。関数の可視性の詳細については、[Functions](./functions)の_Visibility_セクションを参照してください。

## Friend宣言

モジュールは、friend宣言文を使用して他のモジュールをfriendsとして宣言できます。形式は以下の通りです：

- `friend <address::name>` — 以下の例のように完全修飾モジュール名を使用したfriend宣言、または

  ```move
  module 0x42::a {
      friend 0x42::b;
  }
  ```

- `friend <module-name-alias>` — モジュール名エイリアスを使用したfriend宣言。モジュールエイリアスは`use`文で導入されます。

  ```move
  module 0x42::a {
      use 0x42::b;
      friend b;
  }
  ```

モジュールは複数のfriend宣言を持つことができ、すべてのfriendモジュールの和集合がfriendリストを形成します。以下の例では、`0x42::B`と`0x42::C`の両方が`0x42::A`のfriendsと見なされます。

```move
module 0x42::a;

friend 0x42::b;
friend 0x42::c;
```

`use`文とは異なり、`friend`はモジュールスコープでのみ宣言でき、式ブロックスコープでは宣言できません。`friend`宣言は、トップレベル構文（例：`use`、`function`、`struct`など）が許可される場所であればどこにでも配置できます。ただし、可読性のために、friend宣言をモジュール定義の開始近くに配置することをお勧めします。

### Friend宣言ルール

Friend宣言は以下のルールに従います：

- モジュールは自分自身をfriendとして宣言できません。

  ```move
  module 0x42::m { friend Self; // ERROR! }
  //                      ^^^^ モジュール自体をfriendとして宣言することはできません

  module 0x43::m { friend 0x43::M; // ERROR! }
  //                      ^^^^^^^ モジュール自体をfriendとして宣言することはできません
  ```

- Friendモジュールはコンパイラによって認識されている必要があります

  ```move
  module 0x42::m { friend 0x42::nonexistent; // ERROR! }
  //                      ^^^^^^^^^^^^^^^^^ 未バインドモジュール '0x42::nonexistent'
  ```

- Friendモジュールは同じアカウントアドレス内にある必要があります。

  ```move
  module 0x42::m {}

  module 0x42::n { friend 0x42::m; // ERROR! }
  //                      ^^^^^^^ 現在のアドレス外のモジュールをfriendとして宣言することはできません
  ```

- Friends関係は循環モジュール依存関係を作成できません。

  friend関係では循環は許可されません。例えば、`0x2::a` friends `0x2::b` friends `0x2::c` friends `0x2::a`の関係は許可されません。より一般的には、friendモジュールを宣言すると、friendモジュールに対する現在のモジュールの依存関係が追加されます（目的はfriendが現在のモジュールの関数を呼び出すことだからです）。そのfriendモジュールが既に使用されている場合、直接的または推移的に、依存関係の循環が作成されることになります。

  ```move
  module 0x2::a {
      use 0x2::c;
      friend 0x2::b;

      public fun a() {
          c::c()
      }
  }

  module 0x2::b {
      friend 0x2::c; // ERROR!
  //         ^^^^^^ このfriend関係は依存関係の循環を作成します: '0x2::b' は '0x2::a' のfriendで、'0x2::a' は '0x2::c' を使用し、'0x2::c' は '0x2::b' のfriendです
  }

  module 0x2::c {
      public fun c() {}
  }
  ```

- モジュールのfriendリストに重複を含めることはできません。

  ```move
  module 0x42::a {}

  module 0x42::m {
      use 0x42::a as aliased_a;
      friend 0x42::A;
      friend aliased_a; // ERROR!
  //         ^^^^^^^^^ 重複するfriend宣言 '0x42::a'。モジュール内のfriend宣言は一意である必要があります
  }
  ```
