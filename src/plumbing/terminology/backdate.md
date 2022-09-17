# Backdate

回溯：指将在版本 R 中计算的值标记为在某个较早的版本中最后一次更改。

这在有一个较旧的 [memo] M 时完成，并且我们可以比较这两个值来看到它，虽然对 M 的依赖项 ([dependencies])
可能已经改变，但查询函数 ([query function]) 的结果没有改变。

[memo]: ./memo.md
[dependencies]: ./dependency.md
[query function]: ./query_function.md
