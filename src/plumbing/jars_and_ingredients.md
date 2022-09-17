# Jar 和配料

{{#include ../caveat.md}}

本节介绍 Salsa 中的数据是如何组织的，以及 Salsa 条目(items) 之间如何联系起来（例如，依赖条目跟踪）。

## Salsa 条目和配料

Salsa 条目是指可以包括在 Jar 中、带有 Salsa 属性宏标注的条目。例如，跟踪函数是一个 Salsa 条目：

```rust,ignore
#[salsa::tracked]
fn foo(db: &dyn Db, input: MyInput) { }
```

... Salsa 输入也是如此...

```rust,ignore
#[salsa::input]
struct MyInput { }
```

...或跟踪结构：

```rust,ignore
#[salsa::tracked]
struct MyStruct { }
```

每个 Salsa 条目在运行时都需要特定的数据才能运行。这些数据被称为 “**配料**” (ingredients)。

大多数 Salsa 条目只产生一种配料，但有时会产生不止一种配料。

例如，跟踪函数会生成一个 [`FunctionIngredient`]。但跟踪结构会生成几个配料，一个用于结构本身的 
[`TrackedStructIngredient`]，一个用于每个值字段的 [`FunctionIngredient`]。

[`FunctionIngredient`]: https://github.com/salsa-rs/salsa/blob/becaade31e6ebc58cd0505fc1ee4b8df1f39f7de/components/salsa-2022/src/function.rs#L42
[`TrackedStructIngredient`]: https://github.com/salsa-rs/salsa/blob/becaade31e6ebc58cd0505fc1ee4b8df1f39f7de/components/salsa-2022/src/tracked_struct.rs#L18
[`FunctionIngredient`]: https://github.com/salsa-rs/salsa/blob/becaade31e6ebc58cd0505fc1ee4b8df1f39f7de/components/salsa-2022/src/function.rs#L42

### 配料定义了核心逻辑

大多数有趣的 Salsa 代码都存在于这些配料中。例如，当你创建新的跟踪结构，会调用方法 [`TrackedStruct::new_struct`]，它负责确定跟踪结构的 id。

类似地，当你调用一个跟踪函数，它被转换为对 [`TrackedFunction::fetch`] 的调用，从而决定是否有有效的记忆值要返回，或者该函数是否必须执行。

[`TrackedStruct::new_struct`]: https://github.com/salsa-rs/salsa/blob/becaade31e6ebc58cd0505fc1ee4b8df1f39f7de/components/salsa-2022/src/tracked_struct.rs#L76
[`TrackedFunction::fetch`]: https://github.com/salsa-rs/salsa/blob/becaade31e6ebc58cd0505fc1ee4b8df1f39f7de/components/salsa-2022/src/function/fetch.rs#L15

### `Ingredient` trait

每个配料都实现了 [`Ingredient<DB>`] trait ，该 trait 定义了各种配料支持的泛型操作。

例如，可以使用 `maybe_changed_after` 方法来检查自给定版本以来，存储在配料中的某些特定数据是否发生了更改：

下面我们将看到，每个数据库 `DB` 都能够获取一个 `IngredientIndex`，并用它来获取相应的配料 `&dyn Ingredient<DB>`。

这允许数据库对已索引的配料执行泛型操作，而无需确切知道该配料的类型。

[`Ingredient<DB>`]: https://github.com/salsa-rs/salsa/blob/becaade31e6ebc58cd0505fc1ee4b8df1f39f7de/components/salsa-2022/src/ingredient.rs#L15

### Jar 是各种配料的集合

当你声明一个 Salsa 的 jar 时，需列出了该 jar 中包含的每个 Salsa 条目：

```rust,ignore
#[salsa::jar]
struct Jar(
    foo,
    MyInput,
    MyStruct
);
```

这将展开成如下所示的结构体：

```rust,ignore
struct Jar(
    <foo as IngredientsFor>::Ingredient,
    <MyInput as IngredientsFor>::Ingredient,
    <MyStruct as IngredientsFor>::Ingredient,
)
```

`IngredientsFor` trait 用于定义某些 Salsa 条目所需的配料，例如跟踪函数 `foo` 或跟踪结构 `MyInput`。

每个 Salsa 条目都定义了一个类型 `I`，因此 `<I as IngredientsFor>::Ingredient` 提供了 `I` 所需的配料。

### 数据库是一组 Jars

Salsa 的数据库存储最终归结为一组 Jars 结构体，其中每个 Jar 结构体本身包含 Salsa 条目的配料。

因此，数据库可以被认为是一个配料列表，尽管该列表被组织成两级层次结构。

这种两级层次结构的原因是它允许单独的编译和私有化。

罗列 Jars 的 crate 不必知道 Jar 的内容即可将 Jar 结构体嵌入到数据库中。 Jar 中出现的某些类型可能是另一个结构体的私有类型。

### `HasJars` trait 和 `Jars` 类型

每个 Salsa 数据库都实现了 `HasJars` trait，该 trait 由 `salsa::db` 过程宏生成。

此外，`HarJars` trait 还定义了一个映射到 trait 中的 Jars 元组的 `Jars` 关联类型。

例如，给定一个这样的数据库...

```rust,ignore
#[salsa::db(Jar1, ..., JarN)]
struct MyDatabase {
    storage: salsa::Storage<Self>
}
```

`salsa::db` 宏将生成一个包含 `type Jars = (Jar1, ..., JarN)` 的 `HasJars` 实现。

```rust,ignore
{{#include ../../salsa/components/salsa-2022-macros/src/db.rs:HasJars}}
```

反过来说，`salsa::Storage<DB>` 类型最终包含一个嵌入 `DB::Jars` 的 `Shared` 结构体，从而嵌入每个 Jar 的所有数据。

### 配料索引

在初始化期间，数据库中的每个配料都被分配了一个唯一的索引，称为 [`IngredientIndex`]。这是一个 32 位数字，用于标识特定 Jar 中的特定配料。

[`IngredientIndex`]: https://github.com/salsa-rs/salsa/blob/becaade31e6ebc58cd0505fc1ee4b8df1f39f7de/components/salsa-2022/src/routes.rs#L5-L9

### 路线

除了索引，数据库中的每一种配料也有对应的 **路线** (route)。

路线是一个闭包，它在给定一个 `DB::Jars` 元组引用的情况下，返回 `&dyn Ingredient<DB>`。

路线表允许我们从特定配料的 `IngredientIndex` 转到它的 `&dyn Ingredient<DB>` trait 对象。如稍后所述，在初始化数据库时会创建路线表。

### 数据库键和依赖项键

`DatabaseKeyIndex` 标识了存储在特定配料中的特定值。它结合了 [`IngredientIndex`] 与 `key_index`，其中 `key_index` 为 `salsa::Id`：

```rust,ignore
{{#include ../../salsa/components/salsa-2022/src/key.rs:DatabaseKeyIndex}}
```

`DependencyIndex` 类似，但 `key_index` 是可选的。有时当我们希望引用整个配料而不是配料中的任何特定值时，可以使用这种方法。

这些类型的索引用于存储配料之间的联系。例如，每个记录的值都必须跟踪其输入。这些输入被存储为 `DependencyIndex`。

然后我们可以做一些事情，比如通过以下途径知道“这个输入自修订 R 以来是否发生了变化”：

* 使用配料索引查找路线并获取 `&dyn Ingredient<DB>`
* 然后对该 trait 对象调用 `maybe_changed_since` 方法

### `HasJarsDyn`

在上面的设置有一个问题。使用者的代码总是与 `dyn crate::Db` 值交互，其中 `crate::Db` 是 Jar 定义的 trait。

`crate::Db` trait 扩展了 `salsa::HasJar`，而后者又扩展了 `salsa::Database`。

理想情况下，我们应该让 `salsa::Database` 扩展 `salsa::HasJars`，这是访问 Jars 数据的主要 trait 。

但我们不想这样做，因为 `HasJars` 定义了一个关联的类型 `Jars`，这意味着每个对 `dyn crate::Db` 的引用都必须使用类似于 `dyn crate::Db<Jars = J>` 
的内容来指定 Jars 类型。这将是不符合人体工程学的，但更糟糕的是，这实际上是不可能的：最终的 Jars 类型结合了来自多个 crates 的
Jars，因此任何一个单独的 Jar crate 都不知道。

为了解决这个问题，`salsa::Database` 实际上扩展了另一个 trait `HasJarsDyn`，它没有直接显示 `Jars` 
或配料类型，只是提供了可以对配料执行的各种方法，给出了它的 `IngredientIndex`。

像 `Ingredient<DB>` 这样的 trait 需要了解完整的 `DB` 类型。如果一个函数配料直接调用 `Ingredient<DB>`
上的方法，这意味着它必须完全是泛型的，并且只有在整个数据库类型可用时才在最终的 crates 中实例化。

我们通过 `HasJarsDyn` trait 来解决这个问题。`HasJarsDyn` trait 导出一个将“查找配料，调用方法”的步骤合并起来的方法：

```rust,ignore aasaaasdfijjAasdfa
{{#include ../../salsa/components/salsa-2022/src/storage.rs:HasJarsDyn}}
```

因此，从技术上讲，要检查一个输入是否发生了变化，一种配料：

* 在 `dyn Database` 上调用 `HasJarsDyn::maybe_changed_after`
* 该方法由 `#[salsa::db]` 生成实现：
  * 从配料索引中获取配料的路径
  * 使用该路径获取该配料的 `&dyn Ingredient`
  * 在该配料上调用 `maybe_changed_after`

### 初始化数据库

最后要讨论的是数据库是如何初始化的。`Storage<DB>` 的 `Default` 实现是这样的：

```rust,ignore
{{#include ../../salsa/components/salsa-2022/src/storage.rs:default}}
```

首先，它创建一个空的 `Routes` 实例。然后调用 `DB::create_jars` 方法，此方法的实现由 `#[salsa::db]` 宏定义；它只是在每个 Jar 上调用 `Jar::create_jar` 方法：

```rust,ignore
{{#include ../../salsa/components/salsa-2022-macros/src/db.rs:create_jars}}
```

`create_jar` 这个实现由 `#[salsa::jar]` 宏生成，它只是遍历每个 Salsa 条目的所表示的类型，并要求它创建其配料

```rust,ignore
{{#include ../../salsa/components/salsa-2022-macros/src/jar.rs:create_jar}}
```

为任何特定条目创建配料的代码由相关的宏生成（例如 `#[salsa::traced]`、`#[salsa::input]`），但它始终遵循特定的结构。

要创建配料，首先调用 `Routes::push`，它创建指向该配料的路线，并为其分配一个 `IngredientIndex`。然后，调用 `FunctionIngredient::new`
这样的函数来创建结构。到配料的路线被定义为给定 `DB::Jars` 的闭包，该闭包可以找到特定配料的数据。
