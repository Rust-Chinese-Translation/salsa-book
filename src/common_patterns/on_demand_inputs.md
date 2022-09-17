# 按需（惰性）输入

如果你可以轻松地预先提供所有输入，则 Salsa 输入工作得最好。然而，有时输入集是事先不知道的。

一个典型的例子是从磁盘读取文件。虽然 Salsa 可以急切地扫描特定目录并把内存中创建的文件树作为输入结构体，但更直接的方法是延迟读取文件。

也就是说，当某个查询第一次请求文件的文本时：

1. 从磁盘读取文件并缓存它
2. 为此路径设置文件系统监视器 (file-system watcher)
3. 在监视器发送更改通知的时候，更新缓存的文件

这可以在 Salsa 中使用数据库中的缓存输入和增加数据库 trait 方法来从缓存中获得。

一个完整的可运行的文件监视示例见 [lazy-input]。

[lazy-input]: https://github.com/salsa-rs/salsa/tree/master/examples-2022/lazy-input

```rust,ignore
{{#include ../../salsa/examples-2022/lazy-input/src/main.rs:db}}
```

- 在 `Db` trait 上定义一个按需获取 `File` 的方法（它只需要 `&dyn Db` 而不需要 `&mut dyn Db`）
- 每个文件应该只有一个输入结构体，所以给方法实现缓存（`DashMap` 就像 `RwLock<HashMap>` 一样）

然后，执行顶级查询的驱动代码负责在文件更改通知到达时更新文件内容。它更新
Salsa 输入的方式与更新任何其他输入的方式相同。

这里，我们实现了一个简单的驱动循环，每当文件发生变化时，它都会重新编译代码。你可以使用日志来检查是否只重新计算了可能已更改的查询。

```rust,ignore
{{#include ../../../examples-2022/lazy-input/src/main.rs:main}}
```
