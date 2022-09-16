<!-- master#657b856 --->

# 通过回退恢复

在你认为不可能发生循环的情况下，在循环发生时 panic 是可以的。

但有时循环可能是由使用者非法输入引起的，并且无法静态阻止。此时，你可能更喜欢优雅地从循环中恢复，而不是使整个查询陷入 panic。

Salsa 想通过循环恢复 (cycle recovery) 支持这样做。

要使用循环恢复，你可以使用 `#[salsa::recover(my_recover_fn)]` 属性来标注循环中的潜在参与者。

当循环发生时，如果任何参与者 P 有恢复信息，则不会发生 panic。而是中止 P
的执行，并且 P 将执行恢复函数 (recovery function) 以生成其结果。循环中没有恢复信息的参与者使用此恢复结果继续正常执行。

恢复函数具有与查询功能类似的签名，它需要数据库的引用以及描述所发生循环的 `salsa::Cycle`；然后返回查询的结果。示例：

```rust,ignore
fn my_recover_fn(
    db: &dyn MyDatabase,
    cycle: &salsa::Cycle,
) -> MyResultValue
```

`db` 和 `cycle` 参数可用于为使用者准备有用的错误消息。

**重要提示**：尽管恢复函数被赋予了一个 `db` 句柄，但你应该小心避免从恢复内部创建循环或调用可能参与当前循环的查询。尝试这样做可能会导致不一致的结果。

# 找出恢复没有奏效的原因

如果发生循环，并且一些参与者查询有 `#[salsa::recover]` 属性，而其他查询没有，则该查询将被视为不可恢复，并且将直接 panic。

你可以使用 `Cycle::unexpected_participants` 方法找出恢复失败的原因，并添加相应的 `#[salsa::recover]` 属性。
