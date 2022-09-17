# 定义解析器：debug 与测试

作为解析器的最后一部分，我们需要编写一些测试。为此，我们将创建一个数据库，设置输入源文本，运行解析器并检查结果。

然而，在我们能够做到这一点之前，必须先解决一个问题：如何查看像 `Expression` 这样的被 interned 的类型的值呢？

## `DebugWithDb` trait

因为像 `Expression` 这样的 interned 类型只存储一个整数，所以传统的 `Debug` trait 并不是很有用。

要正确打印 `Expression`，你需要访问 Salsa 数据库以找出它的值。

为了解决这个问题，Salsa 提供了一个 `DebugWithDb` trait，它的行为类似于常规的 `Debug`，但以数据库作为参数。

对于实现该 trait 的类型，可以调用 `debug` 方法，该方法将返回一个实现了普通 `Debug` trait 的值，从而你可以编写如下内容：

```rust,ignore
eprintln!("Expression = {:?}", expr.debug(db));
```

然后得到你想要的结果。

对于所有 `#[input]`、`#[interned]` 和 `#[tracked]` 结构体，都会自动派生 `DeugWithDb` trait。

## 转发到普通的 `Debug` trait 

为了保持一致性，有时实现 `DebugWithDb` 是很有用的，即使对于 `Op` 这样只是普通枚举的类型也是如此。你可以这样做：

```rust,ignore
{{#include ../../salsa/examples-2022/calc/src/ir.rs:op_debug_impl}}
```

## 编写单元测试

我们既然已经准备好了 `DebugWithDb` 实现，就可以编写一个简单的单元测试工具了。

下面的 `parse_string` 函数创建一个数据库，设置源文本，然后调用了解析器：

```rust,ignore
{{#include ../../salsa/examples-2022/calc/src/parser.rs:parse_string}}
```

结合 [`expect-test`] crate，我们就可以编写这样的单元测试：

```rust,ignore
{{#include ../../salsa/examples-2022/calc/src/parser.rs:parse_print}}
```

[`expect-test`]: https://crates.io/crates/expect-test
