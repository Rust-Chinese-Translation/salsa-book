<!-- master#68cb5e9 --->

# LRU

[`set_lru_capacity`] 方法可用于将查询的最大容量固定为特定数量的值。

如果在设置之后添加了更多的值，Salsa 则删除较旧的 [memos] 中的值以节省内存。

然而，Salsa 总是保留这些 memo 的依赖项 ([dependency]) 信息，以便仍然可以计算值是否已经改变，即使不知道该值是什么。

[`set_lru_capacity`]: https://docs.rs/salsa/0.16.1/salsa/struct.QueryTableMut.html#method.set_lru_capacity
[memos]: ./memo.md
[dependency]: ./dependency.md
