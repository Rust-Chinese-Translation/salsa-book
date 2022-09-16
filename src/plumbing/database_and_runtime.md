<!-- master#657b856 --->

# 数据库和运行时

Salsa 数据库结构体通过 `#[salsa::db]` 声明。它包含程序执行所需的所有数据：

```rust,ignore
#[salsa::db(jar0...jarn)]
struct MyDatabase {
    storage: Storage<Self>,
    maybe_other_fields: u32,
}
```

该数据分为两类：

* Salsa 管理和存储的数据，包含在 `Storage<Self>` 中。此数据为必须的。
* 使用者定义的其他字段（如 `maybe_other_fields`）。这可以是任何东西，让你访问特殊资源或其他任何资源。

## 并行句柄

当跨并行线程使用时，使用者定义的数据库类型必须支持“快照” (snapshot) 操作。

此快照应创建可供并行线程使用的数据库的克隆。

`Storage` 操作本身支持 `snapshot`。 `Snaphot`方法返回 `Snapshot<DB>` 类型，防止通过 `&mut` 访问这些克隆。

## `Storage`

Salsa 的 `Storage` 结构体包含 Salsa 本身使用和处理的所有数据。有三个关键：

* `Shared` 结构体：包含跨所有快照存储的数据。数据主要是 [Jars 和配料] 一章中描述的配料，但也包含一些同步信息
  (cond var)。这是用于取消 (cancellation) 的，如下所述。
  * `Shared` 中的数据只有在其他线程处于活动状态时才会跨线程共享。
  * 一些操作，如改变输入，需要 `Shared` 的 `&mut` 句柄。这是通过 `Arc::get_mut`
    方法获得的；显然，只有当所有快照和线程都停止执行时，这才是可能的，因为必须只有一个指向 `arc`的句柄。
* `Routes` 结构体包含查找任何特定配料的信息，这也在所有句柄之间共享，其构造也在 [Jars 和配料] 一章中描述了。`Routes` 从 `Shared`
  中分离出来，是因为它在任何时候都是真正不可变的，并且我们希望能够在获得对 `Shared` 的 `&mut` 访问权限的同时持有 `Routes` 的句柄。
* `Runtime` 结构体特定于具体的数据库实例。它包含单个活动线程的数据，以及一些指向其自身的共享数据。

[Jars 和配料]: ./jars_and_ingredients.md

## 修订计数递增和获取 Jars 的可变性

Salsa 的一般模型是，数据库只有一个“主”副本，可能还有多个快照。快照不是直接拥有的，而是装在只允许 `&`-deref 的
`Snapshot<DB>` 类型中，因此唯一可以使用 `&mut`-ref 访问的数据库是主数据库。然而，每个快照只在存储配料的 `Storage` 中的 `Arc` 上有另一个句柄。

无论何时使用者尝试执行 `&mut` 操作，例如修改输入字段，都需要首先取消所有并行快照，并等待这些并行线程完成。快照完成后，我们可以使用
`Arc::get_mut` 获取对配料数据的 `&mut`。这允许我们在没有任何不安全代码的情况下获得 `&mut`
访问权限，并保证已经成功地取消了其他工作线程（否则我们将陷入死锁）。

通过 `jars_mut` 方法获取数据库的 `&mut` 访问权限：

```rust,ignore
{{#include ../../../components/salsa-2022/src/storage.rs:jars_mut}}
```

关键的一点是它在往下之前调用 `cancel_other_workers`：

```rust,ignore
{{#include ../../../components/salsa-2022/src/storage.rs:cancel_other_workers}}
```

## Salsa 运行时

Salsa 运行时提供了被配料访问的帮助方法。例如，它跟踪活动的查询堆栈，并包含用于在查询之间添加依赖关系的方法（如 
`report_track_read`）或[处理循环](./cycles.md)。它还跟踪当前修订版本以及有关持久性低或高的值上次更改的时间的信息。

基本上，配料结构体存储“静态数据” —— 就像记忆中的值 —— 还存储“每个配料”的内容。

运行时存储“活动的、正在进行的”数据，例如哪些查询在栈上，和/或当前活动的查询访问的依赖项。
