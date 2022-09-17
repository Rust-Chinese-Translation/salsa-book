# 什么是 Salsa

<img src="https://raw.githubusercontent.com/salsa-rs/logo/main/FerrisSalsa4-01.svg" alt="" width="300"/>

Salsa 是一个用于编写增量 (incremental) 、按需 (on-demand) 程序的 Rust 框架 ——
这些程序希望适应其输入的变化，持续地产生最新的输出。

Salsa 以 rustc 构建的增量重新编译技术为基础，许多（但不是所有）使用 Salsa
的人都把它用来构建编译器或其他类似的工具。

如果你想了解更多关于 salsa 的信息，请查看：

* [概述](./overview.md)：一个简要的总结
* [教程](./tutorial.md)：一个详细的说明
* 一些[视频](./videos.md)，尽管其中的内容已经过时

如果你想讨论 Salsa，或者想要贡献，请转到我们的 [Zulip](https://salsa.zulipchat.com/)。

> 译者注：
> 
> [Salsa](https://github.com/salsa-rs/salsa) 作者为 Niko Matsakis， [Rust-Analyzer][RA] 和 [apollo-rs 编译器][apollo-rs] 都使用了它。
> 
> 注意：这本书跟随目前正在设计的 Salsa2022（代码位于 [components](https://github.com/salsa-rs/salsa/tree/master/components) 文件夹下，而不是 src 文件夹下），此外
> * 很多篇章仍然需要更新（Niko 为此正在提交一个 [PR](https://github.com/salsa-rs/salsa/pull/396)）
> * [docs.rs](https://docs.rs/salsa/latest/salsa/) 上的文档不是 2022 版，最好自己使用 `cargo doc --no-deps --document-private-items --workspace` 进行本地构建

[RA]: https://github.com/rust-lang/rust-analyzer/blob/c6c0ac26456093b71837e20b0ff51655e0c230f7/docs/dev/guide.md#salsa
[apollo-rs]: https://github.com/apollographql/apollo-rs/blob/632eda9abb3438786ecbad8ecf9e17cb99ac548c/crates/apollo-compiler/README.md#usage
