# Untracked dependency

未跟踪的依赖项 (untracked dependency)：指派生查询 ([derived query]) 的结果依赖于 Salsa 数据库不可见的内容。
* 通过调用 [`report_untracked_read`] 或 [`report_synthetic_read`] 来创建
* 当存在未跟踪的依赖时，如果持久性检查失败，则始终重新执行派生查询
* 更多详细信息，见 [fetch] 的操作说明

[`report_untracked_read`]: https://docs.rs/salsa/0.16.1/salsa/struct.Runtime.html#method.report_untracked_read
[`report_synthetic_read`]: https://docs.rs/salsa/0.16.1/salsa/struct.Runtime.html#method.report_synthetic_read
[derived query]: ./derived_query.md
[derived queries]: ./derived_query.md
[fetch]: ../fetch.md#derived-queries
