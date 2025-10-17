# 可視性修飾子

すべてのモジュールメンバーには可視性があります。デフォルトでは、すべてのモジュールメンバーは_プライベート_です - つまり、定義されたモジュール内でのみアクセス可能です。しかし、可視性修飾子を追加することで、モジュールメンバーを_パブリック_（モジュール外部で見える）、_public(package)_（同じパッケージ内のモジュールで見える）、または_エントリ_（トランザクションから呼び出せるが他のモジュールからは呼び出せない）にできます。

## 内部可視性

モジュールで定義された関数や構造体で可視性修飾子がないものは、そのモジュールに対して_プライベート_です。他のモジュールから呼び出すことはできません。

```move
module book::internal_visibility;

// この関数は同じモジュール内の他の関数から呼び出せる
fun internal() { /* ... */ }

// 同じモジュール -> internal()を呼び出せる
fun call_internal() {
    internal();
}
```

以下のコードはコンパイルされません：

<!-- TODO: add failure flag to example -->

```move
module book::try_calling_internal;

use book::internal_visibility;

// 異なるモジュール -> internal()を呼び出せない
fun try_calling_internal() {
    internal_visibility::internal();
}
```

構造体のフィールドがMoveから見えないからといって、その値が機密に保たれるわけではありません &mdash; Move外部からオンチェーンオブジェクトの内容を読み取ることは常に可能です。暗号化されていないシークレットをオブジェクト内に保存すべきではありません。

## パブリック可視性

構造体や関数は、`fun`または`struct`キーワードの前に`public`キーワードを追加することで_パブリック_にできます。

```move
module book::public_visibility;

// この関数は他のモジュールから呼び出せる
public fun public_fun() { /* ... */ }
```

パブリック関数は他のモジュールからインポートして呼び出すことができます。以下のコードはコンパイルされます：

```move
module book::try_calling_public;

use book::public_visibility;

// 異なるモジュール -> public_fun()を呼び出せる
fun try_calling_public() {
    public_visibility::public_fun();
}
```

一部の言語とは異なり、構造体のフィールドをパブリックにすることはできません。

## パッケージ可視性

_パッケージ_可視性を持つ関数は、同じパッケージ内の任意のモジュールから呼び出せますが、他のパッケージのモジュールからは呼び出せません。つまり、パッケージに対して_内部_です。

```move
module book::package_visibility;

public(package) fun package_only() { /* ... */ }
```

パッケージ関数は同じパッケージ内の任意のモジュールから呼び出せます：

```move
module book::try_calling_package;

use book::package_visibility;

// 同じパッケージ `book` -> package_only()を呼び出せる
fun try_calling_package() {
    package_visibility::package_only();
}
```

## ネイティブ関数

[フレームワーク](./../programmability/sui-framework)と[標準ライブラリ](./standard-library)の一部の関数は`native`修飾子でマークされています。これらの関数はMove VMによってネイティブに提供され、Moveソースコードに本体を持ちません。native修飾子について詳しくは、[Move Reference](./../../reference/functions?highlight=native#native-functions)を参照してください。

```move
module std::type_name;

public native fun get<T>(): TypeName;
```

これは`std::type_name`からの例です。このモジュールについて詳しくは[リフレクション章](./type-reflection)をご覧ください。

## 参考文献

- Move Referenceの[Visibility](./../../reference/functions#visibility)。
