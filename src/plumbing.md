<!-- master#657b856 --->

# 管道

{{#include caveat.md}}

本章记录了 Salsa 生成的代码及其“内部工作原理”，这被称为“管道” (plumbing)。

## 概述

管道分为以下几章：

* The [jars and ingredients](./plumbing/jars_and_ingredients.md) covers how each salsa item (like a tracked function) specifies what data it needs and runtime, and how links between items work.
* The [database and runtime](./plumbing/database_and_runtime.md) covers the data structures that are used at runtime to coordinate workers, trigger cancellation, track which functions are active and what dependencies they have accrued, and so forth.
* The [query operations](./plumbing/query_ops.md) chapter describes how the major operations on function ingredients work. This text was written for an older version of salsa but the logic is the same:
  * The [maybe changed after](./plumbing/maybe_changed_after.md) operation determines when a memoized value for a tracked function is out of date.
  * The [fetch](./plumbing/fetch.md) operation computes the most recent value.
  * The [derived queries flowchart](./plumbing/derived_flowchart.md) depicts the logic in flowchart form.
  * The [cycle handling](./plumbing/cycles.md) handling chapter describes what happens when cycles occur.
* The [terminology](./plumbing/terminology.md) section describes various words that appear throughout.
