# “红-绿”算法

本页面介绍基本的 Salsa 增量算法。这种算法被称为“红-绿”算法，这是 Salsa 这个名字的由来[^salsa-name]。

[^salsa-name]: 译者注：墨西哥辣番茄酱 (salsa) 以鲜艳的红绿配色为特色。

### 数据库的修订版本

Salsa 数据库始终跟踪一个修订版本 (revision)。每次设置输入时，修订都是增量的。

所以 Salsa 从修订版本 `R1` 开始，但当调用 `set` 方法时，将变为 `R2`，然后是 `R3`，依此类推。对于每个输入，Salsa 还跟踪上次更改它的修订版本。

### 基本规则：输入发生变化时重新执行

当你调用跟踪函数时，除了存储已经被返回的值之外，Salsa 还跟踪它所依赖的其他跟踪函数，以及它们的值最后一次更改的修订版本。

当你再次调用该函数时，如果数据库处于新的修订版本中，Salsa 则检查该函数的任何输入是否在该新修订版本中发生了更改。

如果没有发生更改，Salsa 可以只返回其缓存的值。但如果输入已更改（或可能已更改），Salsa 将重新执行该函数以得到最新的结果。

下面是一个简单的例子，其中 `parse_module` 函数调用 `module_text` 函数：

```rust,ignore
#[salsa::tracked]
fn parse_module(db: &dyn Db, module: Module) -> Ast {
    let module_text: &String = module_text(db, module);
    Ast::parse_text(module_text)
}

#[salsa::tracked(return_ref)]
fn module_text(db: &dyn Db, module: Module) -> String {
    panic!("text for module `{module:?}` not set")
}
```

如果调用了两次 `parse_module`，但是其间更改了模块的文本，那么 Salsa 将不得不重新执行 `parse_module`：

```rust,ignore
module_text::set(
    db,
    module,
    "fn foo() { }".to_string(),
);
parse_module(db, module); // executes

// ...some time later...

module_text::set(
    db,
    module,
    "fn foo() { /* add a comment */ }".to_string(),
);
parse_module(db, module); // executes again!
```

### 追溯：有时我们可以更聪明

不过，跟踪函数通常不直接依赖于输入。相反，它们将依赖于其他一些跟踪函数。例如，我们可能有一个读取 AST 的 `type_check` 函数：

```rust,ignore
#[salsa::tracked]
fn type_check(db: &dyn Db, module: Module) {
    let ast = parse_module(db, module);
    ...
}
```

如果模块中文本被更改，Salsa 必须重新执行 `parse_module`，但对源文本的许多更改仍然会生成相同的 AST。

例如，假设我们只添加了一条注释？在这种情况下，如果再次调用 `type_check`，我们将：

* 首先重新执行 `parse_module`，因为它的输入发生了变化。
* 然后比较得到的 AST：如果它与上次相同，我们可以追溯 (backdate) 结果；也就是说，即使输入改变了，输出也没有改变。

## 持久性：优化

作为一种优化，Salsa 包含了持久性 (durability) 的概念，这是指一些被跟踪的数据更改的频率。

例如，在编译 Rust 程序时，你可能会把来自 crates.io 
的输入标记为高持久性输入，因为它们不太可能更改；把当前工作区可以标记为低持久性，因为对其的更改一直在发生。

当设置跟踪函数值时，你还可以给定持久性：

```rust,ignore
module_text::set_with_durability(
    db,
    module,
    "fn foo() { }".to_string(),
    salsa::Durability::HIGH
);
```

对于每个持久性，Salsa 跟踪与该持久性相关的某些输入所发生更改的修订。

如果一个跟踪函数只（传递地）依赖高持久性输入，而你更改了一个低持久性输入，那么 Salsa 
可以非常容易地确定跟踪函数的结果仍然有效，从而避免了逐个遍历输入。
