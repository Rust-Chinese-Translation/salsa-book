<!-- master#68cb5e9 --->

# fetch

```rust,no_run,noplayground
{{#include ../../../src/plumbing.rs:fetch}}
```

`fetch` 方法计算查询的值。最好在可能的情况下重复使用已记忆的值。

## 输入查询

输入查询只是从表中加载结果。

## Interned 查询

Interned 查询将输入映射到 hashmap 中以查找现有的整数。如果不存在，则会创建一个新值。

## Derived queries

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
* 假设依赖项已经被修改或者 memo 不包含被记忆的值：
  * 则执行使用者的查询功能
  * 然后，计算被记忆的值最后被更改的修订版本：
    * 回溯：如果此前被记忆的值存在，并且新值等于该旧值，则回溯 memo，这意味着使用以前的 `changed_at` 修订
      * 由于回溯，查询的依赖可能在某个修订 R1 中改变，但是查询的结果在 R2 先于 R1 的某个修订中改变
    * 否则，使用当前版本
  * 为新值构造一个 memo 并返回它

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

