<!-- master#657b856 --->

# Salsa 调参

## LRU 缓存

你可以为任何非输入查询指定 LRU 缓存大小：

```rust,ignore
let lru_capacity: usize = 128;
base_db::ParseQuery.in_db_mut(self).set_lru_capacity(lru_capacity);
```

默认为 `0`，表示完全禁用 LRU 缓存。

请注意，旧查询的键和结果没有垃圾回收，因此 LRU 缓存是当前唯一可用来避免在 Salsa 上构建长时间运行应用程序使用无限内存的旋钮。

## Intern 查询

[`Arc`]: https://doc.rust-lang.org/std/sync/struct.Arc.html

Intern 查询可以降低查找键的成本，节省内存，并避免使用 [`Arc`]。

对于涉及嵌套的树状数据结构的查询，Intern 尤其有用。

参考：

[`compiler`]: https://github.com/salsa-rs/salsa/blob/master/examples/compiler/main.rs

* [`compiler`] 例子，它使用了 Intern

## 增量的精细程度

参考：
* [模式：选择](./common_patterns/selection.md)
* [`selection`](https://github.com/salsa-rs/salsa/blob/master/examples/selection/main.rs) 例子

## 取消

并发写入或依赖项更改引发的不再需要的查询会被 Salsa 取消。

中间查询的每次访问都是一个潜在的取消点 (cancellation)。取消是通过 panic 来实现的，而 Salsa 的内部设计是为了防止 panic。

如果你的查询包含一个不执行任何中间查询的长循环，则 Salsa 将无法自动取消它。你可能希望通过调用 `db.unwind_if_cancelled()` 来检查取消。

有关取消的更多详细信息，请参考 Salsa 仓库中的取消行为测试。
