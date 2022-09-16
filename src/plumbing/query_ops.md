<!-- master#657b856 --->

# 查询操作

每个查询存储结构体都实现了 [`plumbing`] 模块中的 `QueryStorageOps` trait：

```rust,no_run,noplayground
{{#include ../../salsa/src/plumbing.rs:QueryStorageOps}}
```

它定义了所有查询支持的基本操作。最重要的是这两点：

* [maybe changed after](./maybe_changed_after.md)：如果给定键的查询的值可能在给定的修订版本之后已经发生改变，则返回 true 
* [Fetch](./fetch.md)：返回给定 K 的最新值（或者在不可恢复的循环中返回错误）

[`plumbing`]: https://github.com/salsa-rs/salsa/blob/master/src/plumbing.rs

