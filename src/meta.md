<!-- master#68cb5e9 --->

# 关于这本书本身

## 链接策略

我们试图避免那些容易变得脆弱的链接。

**做**：

* 链接到 `docs.rs` 类型以记录公共接口，但把链接改成 `latest` 版本。
* 链接到源代码中的模块。
* 创建命名锚 ([named anchors]) 来直接嵌入源代码。

[named anchors]: https://rust-lang.github.io/mdBook/format/mdbook.html?highlight=ANCHOR#including-portions-of-a-file

**不要做**：

* 直接链接到 Github 上的行，即使是在特定的提交中，除非你试图引用一段历史代码（“事情在当时是怎样的”）。
