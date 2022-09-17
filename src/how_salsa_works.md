# Salsa 如何工作

## 视频介绍

关于 Salsa 工作原理的最完整介绍，请查看 youtube 上的 [How Salsa Works] 视频。

如果想更深入了解， [Salsa In More Depth] 深入了增量算法的细节。

> 如果你在中国，可以在 B 站上观看 ["How Salsa Works"](https://www.bilibili.com/video/BV1Df4y1A7t3/) 和 ["Salsa In More Depth"](https://www.bilibili.com/video/BV1AM4y1G7E4/)。

[How Salsa Works]: https://youtu.be/_muY4HjSqVw
[Salsa in more depth]: https://www.youtube.com/watch?v=i_IhACacPRY

## 核心理念

Salsa 的关键思想是将你的程序定义为一组查询。每个查询都像函数 `K -> V` 一样使用，该函数从某个 `K`
类型的键映射到一个 `V` 类型的值。查询有两种基本类型：

* 输入：系统的基本输入。你可以随时更改它们。
* 函数：将你的输入转换为其他值的纯函数（没有副作用）。查询的结果被记录下来，以避免大量的重新计算。  
  当你对输入进行更改时，Salsa 将（相当智能地）计算出何时可以重新使用这些被记录的值，以及何时必须重新计算它们。

## 三步使用 Salsa

使用 Salsa 就像 1、2、3 一样简单：

1. 定义一个或多个包含所需输入和查询的查询组 (query groups) 。从一个这样的组开始，但稍后你可以使用多个组将你的系统分解为组件（或将代码分散到多个 crates 中）。
2. 在适当的地方定义查询函数 (query functions)。
3. 定义数据库 (database)，该数据库存储你使用的所有输入/查询。查询结构将体存储所有输入/查询，还可能包含代码需要的任何其他内容（例如，配置数据）。

要查看这方面的实际示例，请查看 [`hello_world`] 示例，该示例有许多解释如何工作的注释。

[`hello_world`]: https://github.com/salsa-rs/salsa/blob/master/examples/hello_world/main.rs

## 深入管道

查看管道章节 ([plumbing]) ，了解对 Salsa 生成的代码以及它如何连接到 Salsa 库的更深入的解释。

[plumbing]: plumbing.md
