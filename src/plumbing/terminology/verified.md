<!-- master#68cb5e9 --->

# Verified

如果 Salsa 已经检查到 [memo] 的值仍然是最新的（即，如果重新执行 [query function] 会保证得到相同的结果），则 memo 在修订版 R 中被验证。

每个 memo 跟踪其最后一次验证的版本，以避免重复检查依赖项在 [fetch] 及 [maybe changed after] 操作期间是否已更改，。

[query function]: ./query_function.md
[fetch]: ../fetch.md
[maybe changed after]: ../maybe_changed_after.md
[memo]: ./memo.md
