# 定义 IR

在定义[解析器]之前，我们需要定义用于 `calc` 程序的 IR (Intermediate Representation， 中间表示)。

在[基本结构]中，我们定义了一些像 `Statement` 和 `Expression` 这样的“伪 Rust”结构；现在我们将它们真的定义它们。

[解析器]: ./parser.md
[基本结构]: ./structure.md
[Jar]: ./jar.md

## Salsa Structs

除了常规的 Rust 类型外，我们还将使用各种 Salsa 结构体。

Salsa 结构体是使用一种 Salsa 属性宏进行了标注的结构体：

* [`#[salsa::input]`](#input-struct)：用于指定计算的“基本输入”
* [`#[salsa::tracked]`](#tracked-structs)：用于指定在计算过程中创建的中间值
* [`#[salsa::interned]`](#interned-structs)：用于指定易于进行相等比较的小型值

所有 Salsa 结构体都将其字段的实际值存储在 Salsa 数据库中。这使我们能够跟踪这些字段的值何时更改，以确定需要重新执行哪些工作。

当你使用上述一种 Salsa 属性标注结构体时， Salsa 实际上会生成一组代码来将该结构链接到数据库。

此代码必须连接到某个 [Jar]。默认为 `crate::Jar`，但你可以使用 `jar=` 属性指定不同的 Jar（如 `#[salsa::input(Jar=Myjar)]`）。

你还必须在 Jar 定义中列出该结构，否则会出现错误。

## `#[input]` struct

我们要定义的第一件事是输入。每个 Salsa 程序都有一些基本的输入来驱动其余的计算。

程序的其余部分必须是这些基本输入的某个确定性函数，这样当这些输入发生变化时，我们可以尽量高效地重新计算该函数的新结果。

把带有 `#[salsa::input]` 属性的 Rust 结构体叫做输入 (input)：

```rust
{{#include ../../salsa/examples-2022/calc/src/ir.rs:input}}
```

我们的编译器中只有一个简单的输入，即 `SourceProgram`，它有一个字段 `text` （字符串）。

### 数据存于数据库中

尽管 Salsa 结构体像其他 Rust 结构体一样被声明，但它的实现方式截然不同。

Salsa 结构体的字段的值存储在 Salsa 数据库中，其本身只包含一个数字标识符。

这意味着结构体实例是复制的（无论它们包含什么字段）。创建结构体的实例和访问字段是通过调用
`new` 以及 getters 和 setter 等方法来完成的。

更具体地说，`#[salsa::input]` 属性将为 `SourceProgram` 生成一个新结构体，如下所示：

```rust
#[derive(Copy, Clone, PartialEq, Eq, PartialOrd, Ord, Hash)]
pub struct SourceProgram(salsa::Id);
```

它还会生成一个方法 `new`，让你在数据库中创建一个 `SourceProgram`。对于输入，需要 `&mut db`，以及每个字段的值：

```rust
let source = SourceProgram::new(&mut db, "print 11 + 11".to_string());
```

你可以使用 `soure.text(&db)` 来读取该字段的值，并使用 `soure.set_text(&mut db，"print 11 * 2".to_string())` 来设置该字段的值。

### 数据库修订

如[概述](../overview.html#salsa-的目标)中所述，当一个函数需要数据库的 `&mut` 时，这意味着它只能从程序增量部分之外调用。

当你更改输入字段的值时，数据库中的“修订计数器” (revision counter) 
会递增，这表明某些输入现在不同了。数据库的“修订” (revision) 指输入值更改之间的数据库状态。

### 表示解析后的程序

接下来，我们将定义一个 **跟踪结构体** (tracked struct) 。输入表示计算的开始，而跟踪结构体表示在计算期间创建的中间值。

在本例中，解析器将接收 `SourceProgram` 结构体，并返回一个表示完全解析过的程序 `Program`：

```rust
{{#include ../../salsa/examples-2022/calc/src/ir.rs:program}}
```

与输入一样的是，跟踪结构体的字段也存储在数据库中。

与输入不同的是，这些字段的值是不变的，它们不能被“set”， Salsa 会在不同的修订版本之间比较它们，以了解它们何时发生了更改。此时，如果解析输入产生相同的 
`Program` 结果（例如，输入时只修改了一些尾随的空格），那么接下来的计算将不需要重新执行。（我们将来会在 IR 中重新讨论跟踪结构体在更多复用中的作用。）

除了字段的值是不可变之外，使用跟踪结构体的 API 非常类似于输入：

* 使用 `new` 来创建一个新值，但是对于跟踪结构体，你只需要 `&dyn Db`，而不需要 `&mut Db`（例如，`Program::new(&db, some_staements)`）
* 使用一个 getter 来读取一个字段的值，就像使用一个输入（例如，`my_func.statements(db)` 读取 `statements` 字段）一样
  * 该字段若被标记为 `#[return_ref]`，则意味着 getter 将返回一个 `&Vec<Statement>`，而不是克隆向量

## 表示函数

我们还将使用跟踪结构体来表示每个函数：定义一个跟踪结构体 `Function` 来表示用户定义的每个函数：

```rust
{{#include ../../salsa/examples-2022/calc/src/ir.rs:functions}}
```

与输入一样的是，跟踪结构体的字段也存储在数据库中。

与输入不同的是，这些字段的值是不变的，它们不能被“set”)， Salsa 会在不同的修订版本之间比较它们，以了解它们何时发生了更改。

例如，如果创建一些 `Function` 的实例 `f`，你可能会发现 `f.body` 字段发生了变化，因为用户更改了 `f` 的定义。这意味着我们必须重新执行依赖于
`f.body` 的那部分代码（但不必重新执行依赖于其他函数的那部分代码）。

除了字段的值是不可变之外，使用跟踪结构体的 API 非常类似于输入：

* 使用 `new` 创建一个新值，但是使用跟踪结构体，你只需要 `&dyn Db`，而不需要 `&mut Db`（例如，`Function::new(&db, some_name, some_args, some_body)`）
* 使用 getter 来读取一个字段的值，就像使用输入（例如，`my_unc.args(db)` 读取 `args` 字段）

### `#[id]` 字段

为了更好地在不同的修订版本间复用，特别是在重新排序时，你可以用 `#[id]` 标记一些实体字段。通常，你在表示实体名称的字段上标注它。

这表明，在两个修订版本 R1 和 R2 中，如果使用相同的名称创建两个函数，则它们引用相同的实体，从而比较它们的其他字段是否相等，以确定需要重新执行什么。

添加 `#[id]` 属性是一种优化，不会影响正确性。有关更多详细信息，请参考[算法][algorithm]页面。

[algorithm]: ../reference/algorithm.md 

## `#[interned]` struct

最后一种 Salsa 结构体是 **驻留结构体** (interned struct)。

与输入和跟踪结构一样，驻留结构体的数据存储在数据库中，你只需传递一个整数。

与这些结构体不同的是，如果将相同的数据实化两次，则会得到相同的整数。

“驻留” 的典型用法是应用于函数名和变量之类的小字符串。

传递必须克隆的 `String` 值的名称既烦人又低效；必须通过字符串比较来比较它们是否相等也是低效的。

因此，我们定义了两个驻留结构体 `FunctionId` 和 `VariableId`，每个结构体都有一个存储字符串的字段：

```rust
{{#include ../../salsa/examples-2022/calc/src/ir.rs:interned_ids}}
```

例如，当你调用 `FunctionId::new(&db, "my_string".to_string())` 时，它将返回一个 `FunctionId` —— 
它只是一个 Newtype 的整数。但是，如果再次调用相同参数的 `new`，则会返回相同的整数：

```rust
let f1 = FunctionId::new(&db, "my_string".to_string());
let f2 = FunctionId::new(&db, "my_string".to_string());
assert_eq!(f1, f2);
```

### 驻留结构体中的 id

驻留结构体中的 id 保证在一个修改版本内保持一致，但不会在不同版本之间保持一致（但你不必关心）。

它保证不会在单个版本内更改，因此你的程序中所驻留的东西会得到一致的结果。

然而，当你更改输入时， Salsa 可能会选择清除一些中间值并选择不同的整数。

然而，如果发生这种情况，它还将确保重新执行每个驻留该值的函数，以便所有这些函数仍然得到一个一致的值，只是与它们在以前的修订版本中看到的值不同。

换句话说，在 Salsa 计算期间，你可以假设驻留后生成一个一致的整数，并且你不必考虑它。

但是，如果在计算之外导出（保存）被驻留的标识符，然后更改输入，则之前导出的东西可能不再有效或可能引用不同的值。

### 表达式和语句

我们不会对表达式和语句使用任何特殊的“salsa 结构”：

```rust
{{#include ../../salsa/examples-2022/calc/src/ir.rs:statements_and_expressions}}
```

由于不跟踪语句和表达式，这意味着我们只尝试在函数的颗粒度上获得增量复用 ——
每当函数体中的任何内容发生更改时，我们都会认为整个函数体是脏的，并重新执行依赖于它的任何东西。

像这样划出某种“合理且粗略”的界限通常是有意义的。

这种方法的一个缺点是：我们将位置内联到每个结构体中。
