<!-- master#1363d78 --->

# Jar 和数据库

在定义 Salsa 程序的有趣部分之前，我们必须先定义 Salsa 数据库的结构体。

数据库是一个结构体，它最终存储 Salsa 的所有中间状态，例如来自[跟踪函数][tracked-functions]的被记忆的 (memoized) 返回值。

[tracked-functions]: ../overview.html#tracked-fn

数据库本身是以一些中间结构 (intermediate structure) 的形式定义的，这些中间结构被称为 **jars**[^jar]，并包含每个函数的数据。

这种设置允许 Salsa 程序在多个 crates 中划分功能。

通常，你为每个 crate 定义一个 Jar 结构，然后在构建最终的数据库时，只需列出 Jar 结构。

这让 crates 定义私有函数和其他属于 Jar 结构的内容，但数据库不直接知道这些内容。

[^jar]: 一罐 Salsa 酱。嗯，明白了吗？明白为什么叫 Jar 了吗？[^java]

[^java]: 好吧，也许它也会让人联想到 Java 的 `.jar` 文件，但它们并没有真正的关系。Jar 只是一个 Rust 结构体，而不是一种包装格式。

## 定义 Jar 结构体

给一个元组结构体使用 `#[salsa::jar]` 属性来定义 Jar 结构体：

```rust
{{#include ../../../calc-example/calc/src/main.rs:jar_struct}}
```

虽然这不是必需的，但强烈建议：将 `Jar` 结构放在你的 crate 的根模块，这样它就可以通过 `crate::Jar` 路径使用。

所有其他的 salsa 注释都引用了 Jar 结构体，并且它们都默认 `crate::Jar` 路径。如果你将 Jar 放在其他地方，则必须覆盖该默认值。

## 定义数据库 trait

`#[salsa::jar]` 中还包含 `db = Db` 字段。此字段的值（通常为 `Db` ）代表数据库的 trait 的名称。

Salsa 程序从不直接引用数据库；相反，它们使用 `&dyn Db` 参数。

这可以单独编译：你有一个数据库，其中包含两个 Jars 的数据，但这些 Jars 彼此不依赖。

我们的 `calc` crate 的数据库 trait 非常简单：

```rust
{{#include ../../../calc-example/calc/src/main.rs:jar_db}}
```

当你定义像 `Db` 这样的数据库 trait 时，需要做的一件事是，它必须有一个 supertrait `salsa::DbWithJar<Jar>`，其中 `Jar` 是 Jar 结构体。

如果你的 Jar 依赖于其他 Jar，则可拥有多个这样的超特征（例如， `salsa::DbWithJar<other_crate::Jar>`）。

通常情况下，`Db` trait 没有其他成员或 supertraits，但你也可以自由地在该 trait 中添加任何你想要的其他东西。

当你定义最终的数据库时，它将实现这个 `Db` trait，然后你可以定义这些其他内容的实现。

这让你为 Jar 在需要时从数据库请求上下文或其他信息，而不是通过 Salsa 进行管理。

## 给 Jar 实现数据库 trait

必须给数据库结构体实现 `Db` trait。

我们将在[后面的小节](./db.md)中定义数据库结构体，一种方式是在那里简单地实现 `Db` trait。

然而，由于我们没有在 trait 中定义任何逻辑，一种常见的做法是为实现了 `DbWithJar<Jar>` 的任何类型编写一个
blanket impl，像这样：

```rust
{{#include ../../../calc-example/calc/src/main.rs:jar_db_impl}}
```

## 总结

如果 Jar 的概念对你来说有点抽象，不要想太多。长话短说，它是指在创建 Salsa 程序时，你需要做以下几件事：

* 在每个 crates 中：
  * 定义一个带 `#[salsa::jar(db = Db)]` 的结构体，且通常位于根模块，所以它是 `crate::Jar`，并在其中列出各种带 salsa 注释的东西。
  * 定义一个 `Db` trait，通常为 `crate::Db` ，你将在 memoized 函数和其他地方使用它来引用数据库结构体。
* 只做一次，通常在最后一个 crate 中：
  * 定义一个数据库 `D`，如[下一节](./db.md)所述，它为每个 crate 的 Jar 的列表
  * 给数据库类型 `D` 的每个 Jar 实现 `Db` trait（通常通过 blanket impl 实现）
