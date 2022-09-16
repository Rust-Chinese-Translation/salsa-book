<!-- master#68cb5e9 --->

# Memo

memo 存储有关上次执行某个查询 ([query]) Q 的 [query function] 的信息：

* 通常，它包含从该查询函数返回的值，因此不必再次执行这个查询函数。
  * 然而，这并不总是正确的：一些查询不缓存其结果值，并且值也可能由于 [LRU] 集合而被删除。
  * 此时，memo 仅存储依赖项 ([dependency]) 的信息，该信息对于确定将 Q 作为依赖项的其他查询是否已更改很有用。
* memo 上次验证 ([verified]) 的修订版本 ([revision])。
* memo 的值上次更改的 ([changed at]) 修订版本。注意，它可能是回溯的 ([backdated])。
* memo 的依赖项 ([dependencies]) 的最小持久性 ([durability]) 。
* （如果有的话）完整的依赖项的集合，或 memo 未跟踪的依赖项 ([untracked dependency]) 的标记。

[revision]: ./revision.md
[backdated]: ./backdate.md
[dependencies]: ./dependency.md
[dependency]: ./dependency.md
[durability]: ./durability.md
[untracked dependency]: ./untracked.md
[verified]: ./verified.md
[query]: ./query.md
[query function]: ./query_function.md
[changed at]: ./changed_at.md
[LRU]: ./LRU.md
