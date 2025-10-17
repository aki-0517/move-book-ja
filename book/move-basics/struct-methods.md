# 構造体メソッド

Moveコンパイラは_レシーバー構文_`e.f()`をサポートし、構造体のインスタンスに対して呼び出すことができるメソッドの定義を許可します。「レシーバー」という用語は、メソッド呼び出しを受け取るインスタンスを特に指します。これは他のプログラミング言語のメソッド構文のようなものです。これは構造体のフィールドで動作する関数を定義する便利な方法で、構造体のフィールドへの直接アクセスを提供し、構造体をパラメータとして渡すよりもクリーンで直感的なコードを作成します。

## メソッド構文

関数の最初の引数が、その関数を定義するモジュール内部の構造体である場合、その関数は`.`演算子を使用して呼び出すことができます。しかし、最初の引数の型が別のモジュールで定義されている場合、メソッドはデフォルトでは構造体と関連付けられません。この場合、`.`演算子構文は使用できず、関数は標準的な関数呼び出し構文を使用して呼び出す必要があります。

モジュールがインポートされると、そのメソッドは自動的に構造体と関連付けられます。

```move file=packages/samples/sources/move-basics/struct-methods.move anchor=hero

```

## メソッドエイリアス

メソッドエイリアスは、モジュールが複数の構造体とそのメソッドを定義する際の名前の競合を避けるのに役立ちます。また、構造体により説明的なメソッド名を提供することもできます。

構文は以下の通りです：

```move
// ローカルメソッド関連付け用
use fun function_path as Type.method_name;

// エクスポートされたエイリアス
public use fun function_path as Type.method_name;
```

> パブリックエイリアスは、同じモジュールで定義された構造体に対してのみ許可されます。他のモジュールで定義された構造体に対しては、エイリアスを作成することはできますが、パブリックにすることはできません。

In the example below, we changed the `hero` module and added another type - `Villain`. Both `Hero`
and `Villain` have similar field names and methods. To avoid name conflicts, we prefixed methods
with `hero_` and `villain_` respectively. However, using aliases allows these methods to be called
on struct instances without the prefix:

```move file=packages/samples/sources/move-basics/struct-methods-2.move anchor=hero_and_villain

```

In the test function, the `health` method is called directly on the `Hero` and `Villain` instances
without the prefix, as the compiler automatically associates the methods with their respective
structs.

> Note: In the test function, `hero.health()` is calling the aliased method, not directly accessing
> the private `health` field. While the `Hero` and `Villain` structs are public, their fields remain
> private to the module. The method call `hero.health()` uses the public alias defined by
> `public use fun hero_health as Hero.health`, which provides controlled access to the private
> field.

<!-- ## Aliasing an external module's method

It is also possible to associate a function defined in another module with a struct from the current
module. Following the same approach, we can create an alias for the method defined in another
module. Let's use the `bcs::to_bytes` method from the [Standard Library](./standard-library) and
associate it with the `Hero` struct. It will allow serializing the `Hero` struct to a vector of
bytes.

```move file=packages/samples/sources/move-basics/struct-methods-3.move anchor=hero_to_bytes
``` -->

## Further Reading

- [Method Syntax](./../../reference/method-syntax) in the Move Reference.
