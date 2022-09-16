<!-- master#1363d78 --->

# 定义解析器：报告错误

解析器中的一个有趣的例子是如何处理解析错误。

因为 Salsa 函数是有记忆的，且可能不会执行，所以它们不应该有副作用，所以我们不想只调用 `eprintln!` ——
如果这样做，则只会在第一次调用函数时报告错误，而仅返回被记住的值的后续调用中，`eprintln!` 不会报告错误（这不是我们要的）。

Salsa 定义了一种称为 **累加器** (accumulator) 的管理机制。

在我们的例子中，在 `ir` 模块中定义了一个名为 `Diagnotics` 的累加器结构体：

```rust
{{#include ../../../calc-example/calc/src/ir.rs:diagnostic}}
```

累加器结构体始终是具有单个字段的 newtype 结构体，在本例中为 `Diagnotics` 类型。

记忆函数 (memoized functions) 可以将 `Diagnostic` 的值推送到累加器上。

稍后，你可以调用一个方法来查找由记忆函数或它调用的任何函数推送的所有值（例如，可以获得由 `parse_statements` 函数生成的一组 `Diagnostic` 的值）。

`Parser::report_error` 方法有一个推送诊断的示例：

```rust
{{#include ../../../calc-example/calc/src/parser.rs:report_error}}
```

调用关联函数 `accumulated` 来获得由 `parse_errors` 或任何其他记忆函数生成的诊断集，：

```rust
let accumulated: Vec<Diagnostic> =
    parse_statements::accumulated::<Diagnostics>(db);
                      //            -----------
                      //     Use turbofish to specify
                      //     the diagnostics type.
```

`accumulated` 以数据库 `db` 为参数，并返回 `Vec`。
