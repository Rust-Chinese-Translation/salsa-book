<!-- master#68cb5e9 --->

# Query function

查询函数 (query function)：指使用者提供的函数，Salsa 执行该函数来计算派生查询 ([derived query]) 的值。

Salsa 假定所有查询函数都是它们依赖项 ([dependencies]) 的“纯”函数，除非使用者报告未跟踪的读取 ([untracked read])。

Salsa 总是假设查询函数没有重要的副作用，即它们不会通过网络发送你希望查看其结果的消息，因此它不必重新执行函数，除非它需要它们的返回值。

[derived query]: ./derived_query.md
[dependencies]: ./dependency.md
[untracked read]: ./untracked.md
