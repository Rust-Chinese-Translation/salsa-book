# 定义解析器：存储函数的值和输入

`calc` 编译器的下一步是定义解析器 (parser) 。

解析器的作用是接收 `ProgramSource` 输入，从 `text` 字段读取字符串，并创建我们在 [`ir` 模块][ir]中定义的 `Statement`、`Function` 和 `Expression` 结构。

为了最大限度地减少依赖，我们将编写递归下降解析器 ([recursive descent parser])。

另一种选择是使用 [Rust 解析框架][parsing framework]。

我们不会在本教程中讨论解析本身 —— 如果你想了解它是如何工作的，可以阅读代码。我们将只关注与 Salsa 相关的方面。

[recursive descent parser]: https://en.wikipedia.org/wiki/Recursive_descent_parser
[parsing framework]: https://rustrepo.com/catalog/rust-parsing_newest_1
[ir]: ./ir.md

## `parse_statements` 函数

解析器的起点是 `parse_statements` 函数：

```rust
{{#include ../../salsa/examples-2022/calc/src/parser.rs:parse_statements}}
```

此函数标注了 `#[salsa::tracked]`。这意味着，当它被调用时， Salsa 将跟踪它读取的输入以及返回的值。

返回值被存储下来 (memoized)，这意味着如果再次调用此函数而不更改输入， Salsa 将只克隆结果，而不是重新执行这个函数。

### 跟踪函数是复用的单位

跟踪函数是 Salsa 实现增量复用的核心部分。该框架的目标是避免重新执行跟踪的函数，而是克隆它们的结果。

Salsa 使用[红-绿算法](../reference/algorithm.md)来决定何时重新执行函数。简而言之，如果
- (a)跟踪函数读取已经更改了的输入
- 或者(b)跟踪函数直接调用另一个跟踪函数，并且那个函数的返回值已经更改

则重新执行跟踪的函数。

在 `parse_statements` 的情况下，它直接读取 `ProgramSource::text`，所以如果文本发生变化，那么解析器将重新执行。

通过选择将那些标记了 `#[tracked]` 的函数，你可以控制获得多少复用。

这个例子中，我们选择将最外层的解析函数标记为跟踪的，而不是内层的。

这意味着如果输入发生变化，我们将始终重新解析整个输入并重新创建结果语句，依此类推。

但我们稍后将看到，这并不意味着总是重新运行类型检查器和编译器的其他部分。

这种权衡是有意义的，因为
- (a)解析非常便宜，因此跟踪和启用细粒度复用的开销不会得到回报
- 并且(b)由于字符串只是一个没有任何结构的大二进制字节对象 (blob-o-bytes)，所以很难确定需要重新解析 IR 的哪些部分

有些系统确实选择进行更细粒度的重新解析，通常是通过对字符串执行“第一遍”以给它一点结构，例如识别函数，但将每个函数体的解析推迟到以后。

在 Salsa 中设置这种方案相对容易，并且我们稍后将使用的相同原理来避免重新执行类型检查器。

### 跟踪函数的参数

跟踪函数 (tracked function) 的第一个参数始终是数据库 `db: &dyn crate::db`。它必须是与 Jar 有关的任何数据库的 `dyn` 值。

跟踪函数的第二个参数始终是某种 Salsa 结构。

存储函数 (memoized function) 的第一个参数始终是数据库，它应该是与 Jar 有关的数据库 trait 的值 `dyn Trait`（默认的 Jar 是 `crate::jar`）。

跟踪函数也可能带有其他参数，尽管这里的示例没有。带有附加参数的函数效率较低，灵活性较差。如果可以的话，通常最好将跟踪函数构造为带单个 Salsa 结构的函数。

### `#[tracked(return_ref)]`

你可能已经注意到， `parse_statements` 被标记为 `#[salsa::tracked(return_ref)]`。

通常，当你调用跟踪函数时，你得到的结果会从数据库中克隆出来。而该属性表示返回对数据库的引用。

因此，当调用 `parse_statements` 时，它将返回 `&Vec<Statement>`，而不是克隆 `Vec`。

这对于性能优化很有用。（你可能还记得本教程的 [ir] 中的 `#[return_ref]`，它位于结构体字段上，含义大致相同。）
