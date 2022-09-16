<!-- master#68cb5e9 --->

# 在某个修订之后可能已更改

```rust,no_run,noplayground
{{#include ../../../src/plumbing.rs:maybe_changed_after}}
```

`maybe_changed_after` 计算查询的值在给定的修订后是否发生了更改。换句话说，如果查询 `Q` 的值在修订 `(R+1)..R_now` 中可能已经改变，则
`Q.maybe_changed_after(R)` 为 true，其中 `R_now` 是当前修订。

注意，询问 `maybe_changed_after(R_now)` 是没有意义的。

## 输入查询

输入查询由使用者显式设置。因此，`maybe_changed_after` 可以只检查上次设置该值的时间并进行比较。

## Interned 查询

## 派生查询

派生查询的逻辑更为复杂。这里总结一下高级别的想法，但你可能会发现[流程图][flowchart]有助于深入挖掘。

[术语][terminology]一章也可能很有用；在某些情况下，我们会链接到有关单词首次用法的部分。

* 如果 [memo] 已存在，则检查该备忘录是否在当前修订版本 ([revision]) 中进行了验证 ([verified])。
  * 如果是，则比较它更改所在 ([changed at]) 修订，并适当地返回 true 或 false。
* 如果 memo 不存在，则必须检查依赖项 ([dependencies]) 是否已被修改：
  * 设 R 是上次验证 memo 的修订版本；我们希望知道自修订 R 以来，是否有任何依赖项被更改。
  * 首先，检查持久性 ([durability])。对于每个 memo，跟踪 memo 依赖项的最小持久性。如果 memo 具有持久性 D，并且自上次验证以来，持久性为 D 的输入没有任何更改，则可以认为 memo 已验证，从而无需任何进一步工作。
  * 如果持久性检查不够充分，则必须逐个检查依赖项。为此，需迭代每个依赖项 D，并调用 [maybe_changed_after] 操作以检查自修订版本 R 以来 D 是否已更改。
  * 如果未修改依赖项：将 memo 标记为已验证，并使用其在更改所在的修订以返回 true 或 false。
  * 如果依赖项已修改：
    * 执行使用者的查询函数（与 [fetch] 相同)，这可能会回溯 ([backdate]) 结果值。
    * 比较最终 memo 中更改所在的修订版本，并返回 true或 false。

[flowchart]: ./derived_flowchart.md
[terminology]: ./terminology.md
[memo]: ./terminology/memo.md
[verified]: ./terminology/verified.md
[revision]: ./terminology/revision.md
[changed at]: ./terminology/changed_at.md
[dependencies]: ./terminology/dependency.md
[durability]: ./terminology/durability.md
[maybe_changed_after]: ./maybe_changed_after.md
[changed at]: ./terminology/changed_at.md
[fetch]: ./fetch.md
[backdate]: ./terminology/backdate.md
