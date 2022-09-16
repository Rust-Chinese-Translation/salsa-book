<!-- master#1363d78 --->

# 基本结构

在使用 Salsa 之前，让我们先来讨论一下 calc 编译器的基本结构。

Salsa 所设计的一部分内容是，你能够编写感觉“非常接近”正常 Rust 程序的程序。

## 示例程序

这是我们的示例 calc 程序：

```
x = 5
y = 10
z = x + y * 3
print z
```

## 解析器

calc 编译器接受由字符串表示的程序作为输入：

```rust
struct ProgramSource {
    text: String
}
```

它做的第一件事是将该字符串解析为一系列语句，这些语句看起来类似于下面的伪 Rust 代码[^lexer]：

```rust
enum Statement {
    /// Defines `fn <name>(<args>) = <body>`
    Function(Function),
    /// Defines `print <expr>`
    Print(Expression),
}

/// Defines `fn <name>(<args>) = <body>`
struct Function {
    name: FunctionId,
    args: Vec<VariableId>,
    body: Expression
}
```

其中，表达式类似于这样（伪 Rust 代码，因为 `Expression` 枚举是递归的）：

```rust
enum Expression {
    Op(Expression, Op, Expression),
    Number(f64),
    Variable(VariableId),
    Call(FunctionId, Vec<Expression>),
}

enum Op {
    Add,
    Subtract,
    Multiply,
    Divide,
}
```

最后，对于函数/变量名， `FunctionId` 和 `VariableId` 类型将是被驻留的 (interned) 字符串：

```rust
type FunctionId = /* interned string */;
type VariableId = /* interned string */;
```

[^lexer]: 因为 calc 非常简单，所以我们不必费心将词法分析器 (lexer) 和解析器 (parser) 分开。

## 检查器

“检查器” (checker) 的任务是确保用户只引用已定义的变量。

我们将以一种“无上下文”的风格编写检查器，这有点不直观，但允许更多的增量复用。

其思想是为给定的表达式计算它引用的变量。然后有一个函数 `check`，它确保这些变量是已经定义的变量的子集。

## 翻译器

翻译器 (interpreter) 将执行程序并打印结果。这里不需要太多的增量复用，尽管这当然是可能的。
