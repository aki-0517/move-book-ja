---
title: '条件式 | リファレンス'
description: ''
---

# 条件式 `if`

`if`式は、特定の条件が真の場合にのみコードが評価されることを指定します。
例えば：

```move
if (x > 5) x = x - 5
```

条件は`bool`型の式である必要があります。

`if`式は、条件が偽の場合に評価する別の式を指定するために、オプションで`else`句を含むことができます。

```move
if (y <= 10) y = y + 1 else y = 10
```

「真」の分岐または「偽」の分岐のいずれかが評価されますが、両方ではありません。どちらの分岐も単一の式または式ブロックにすることができます。

条件式は値を生成する可能性があるため、`if`式は結果を持ちます。

```move
let z = if (x < 100) x else 100;
```

`else`句が指定されていない場合、偽の分岐はデフォルトでユニット値になります。以下は同等です：

```move
if (condition) true_branch // 暗黙のデフォルト: else ()
if (condition) true_branch else ()
```

真と偽の分岐の式は互換性のある型である必要があります。例えば：

```move
// xとyはu64整数である必要があります
let maximum: u64 = if (x > y) x else y;

// highlight-error-start
// ERROR! 分岐の型が異なります
let z = if (maximum < 10) 10u8 else 100u64;

// ERROR! 分岐の型が異なります。デフォルトの偽分岐は()でありu64ではありません
let y = if (maximum >= 10) maximum;
// highlight-error-end
```

Commonly, `if` expressions are used in conjunction with
[expression blocks](./../variables#expression-blocks).

```move
let maximum = if (x > y) x else y;
if (maximum < 10) {
    x = x + 10;
    y = y + 10;
} else if (x >= 10 && y >= 10) {
    x = x - 10;
    y = y - 10;
}
```

## Grammar for Conditionals

> _if-expression_ → **if (** _expression_ **)** _expression_ _else-clause_<sub>_opt_</sub> >
> _else-clause_ → **else** _expression_
