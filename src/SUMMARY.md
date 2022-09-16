# Summary

- [关于 salsa](./about_salsa.md)

# 如何使用 Salsa

- [概述](./overview.md)
- [教程： calc 语言](./tutorial.md)
  - [基本结构](./tutorial/structure.md)
  - [Jars 和数据库](./tutorial/jar.md)
  - [定义数据库结构体](./tutorial/db.md)
  - [定义 IR：各种 Salsa 结构体](./tutorial/ir.md)
  - [定义解析器：存储函数的值和输入](./tutorial/parser.md)
  - [定义解析器：报告错误](./tutorial/accumulators.md)
  - [定义解析器：debug 与测试](./tutorial/debug.md)
  - [Defining the checker](./tutorial/checker.md)
  - [Defining the interpreter](./tutorial/interpreter.md)
- [参考](./reference.md)
  - [算法](./reference/algorithm.md)
- [模式](./common_patterns.md)
  - [选择](./common_patterns/selection.md)
  - [按需（惰性）输入](./common_patterns/on_demand_inputs.md)
- [调参](./tuning.md)
- [处理循环](./cycles.md)
  - [通过回退恢复](./cycles/fallback.md)

# Salsa 内部如何工作

- [Salsa 如何工作](./how_salsa_works.md)
- [视频介绍](./videos.md)
- [管道](./plumbing.md)
  - [Jars 和配料](./plumbing/jars_and_ingredients.md)
  - [数据库和运行时](./plumbing/database_and_runtime.md)
  - [查询操作](./plumbing/query_ops.md)
    - [maybe_changed_after](./plumbing/maybe_changed_after.md)
    - [fetch](./plumbing/fetch.md)
    - [派生查询流程图](./plumbing/derived_flowchart.md)
    - [处理循环](./plumbing/cycles.md)
  - [术语](./plumbing/terminology.md)
    - [Backdate](./plumbing/terminology/backdate.md)
    - [Changed at](./plumbing/terminology/changed_at.md)
    - [Dependency](./plumbing/terminology/dependency.md)
    - [Derived query](./plumbing/terminology/derived_query.md)
    - [Durability](./plumbing/terminology/durability.md)
    - [Input query](./plumbing/terminology/input_query.md)
    - [Ingredient](./plumbing/terminology/ingredient.md)
    - [LRU](./plumbing/terminology/LRU.md)
    - [Memo](./plumbing/terminology/memo.md)
    - [Query](./plumbing/terminology/query.md)
    - [Query function](./plumbing/terminology/query_function.md)
    - [Revision](./plumbing/terminology/revision.md)
    - [Salsa item](./plumbing/terminology/salsa_item.md)
    - [Salsa struct](./plumbing/terminology/salsa_struct.md)
    - [Untracked dependency](./plumbing/terminology/untracked.md)
    - [Verified](./plumbing/terminology/verified.md)

# 附录

- [关于这本书本身](./meta.md)
