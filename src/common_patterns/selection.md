<!-- master#657b856 --->

# 选择

“选择 (selection) ”（或“防火墙 (firewall) ”）模式是指当你有一个从其他 Qbase 读取的查询 Qsel，并从 Qsel 返回的 Qbase 中提取一些小信息。

具体地说，Qsel 不组合来自其他查询的值。因此，在某种意义上，Qsel 是多余的 ——
你可以自己从 Qbase 中提取信息，而不需要 Salsa 机制。但 Qsel 的作用在于，它限制了更改 Qbase 时所需的重新执行的数量。

## 示例：基本查询

例如，假设你有一个 `parse` 查询，它解析请求的输入文本，并返回一个 `ParsedResult`，其中包含头信息 (header) 和主体信息 (body)：

```rust,ignore
{{#include ../../salsa/examples/selection/main.rs:request}} 
```

## 示例：选择查询

现在，你有许多派生查询，它们只查看标题。例如，可以提取 `Content-Type` 的 header：

```rust,ignore
{{#include ../../salsa/examples/selection/util1.rs:util1}} 
```

## 为什么更推荐选择查询？

这个 `content_type` 查询是选择模式的一个实例。它只从 `ParsedResult`
中“选择”一小部分信息。你可能根本没有将其作为查询，而是将其作为 `ParsedResult` 上的方法。

但是对 `content_type` 使用查询有一个好处：现在如果有仅依赖于 `content_type`
（或者可能依赖于通过类似模式提取的其他 header）的下游查询，那么当请求改变时，这些查询将不必重新执行，除非
content-type 的 header 改变。考虑以下依赖关系图：

```text
request_text  -->  parse  -->  content_type  -->  (other queries)
```

当 `request_text` 发生变化时， Salsa 总是必须重新执行 `parse`。

* 如果这产生新的解析结果，Salsa 还将重新执行 `content_type`
* 但是如果 `content_type` 的结果没有改变，那么 Salsa 不会重新执行其他查询

## 更多级别的选择

事实上，在我们的示例中，可能会考虑引入另一个级别的选择。与其让 `content_type` 直接访问 `parse` 的结果，不如插入一个只提取 header 的选择查询：

```rust,ignore
{{#include ../../salsa/examples/selection/util2.rs:util2}} 
```

这将产生如下所示的依赖关系图：

```text
request_text  -->  parse  -->  header -->  content_type  -->  (other queries)
```

这样做的好处是，只影响 body 或只消耗请求的一小部分的更改根本不需要重新执行 `content_type`。如果有许多依赖的 headers，这尤其有用。

## 关于克隆和效率

在本例中，我们使用了 `Vec` 和 `String` 等常见的 Rust 类型，并且相当频繁地克隆 (clone) 它们。

这在 Salsa 中会运行得很好，但可能不是最有效的。这是因为每个克隆都将产生结果的深副本 (deep copy)。

一个简单的解决方法是，你可以将在数据结构是使用 `Arc`（例如，`Arc<Vec<ParsedHeader>>`），这会降低克隆成本。
