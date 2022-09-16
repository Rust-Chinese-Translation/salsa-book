<!-- master#1363d78 --->

# 定义数据库结构体

现在我们已经定义了 [Jar](./jar.md)，接下来要创建数据库结构体。

数据库结构体是所有 Jars 聚集在一起的地方。通常，它只由应用程序的“驱动程序” (driver)
使用：启动程序、提供输入的程序和转发输出的程序。

在 `calc` 中，数据库结构体在 [`db` 模块]中，看起来是这样的：

[`db` 模块]: https://github.com/salsa-rs/salsa/blob/master/calc-example/calc/src/db.rs

```rust
{{#include ../../../calc-example/calc/src/db.rs:db_struct}}
```

`#[salsa::db(...)]` 属性把要包含的所有 Jars 作为参数。

该结构体必须有一个名为 `storage` 的字段，其类型为 `salsa::Storage<Self>`，但此外也可以包含你想要的任何其他字段。

`storage` 结构体拥有 `db` 属性中列出的 Jars 的所有数据。

`#[salsa::db(...)]` 属性为我们前面看到的 `salsa::HasJar<crate::Jar>` trait 自动生成一组实现。这意味着要

## 实现 `salsa::Database` trait

除了结构体本身，我们还必须添加一个 `salsa::Database` 实现：

```rust
{{#include ../../../calc-example/calc/src/db.rs:db_impl}}
```

## 实现 `salsa::ParallDatabase` trait

如果你想要允许同时从多个线程访问数据库，那么你还需要实现 `ParallDatabase` trait：

```rust
{{#include ../../../calc-example/calc/src/db.rs:par_db_impl}}
```

## 实现 `Default` trait

这不是必需的，但实现 `Default` 通常是让使用者实例化你的数据库的一种便捷方式：

```rust
{{#include ../../../calc-example/calc/src/db.rs:default_impl}}
```

## 给每个 Jar 实现 traits

`Database` 结构体还需要让每个 Jar 实现数据库 trait。

不过，在我们的例子中，已经[通过 blanket impl] 为做到了，因此不需要执行任何操作。

这是推荐的做法，除非你的 trait 依赖于 `Database` 本身的字段的自定义成员（例如，有时 `Database` 包含你想要访问的某种自定义资源）。

[通过 blanket impl]: jar.html#给-jar-实现数据库-trait
