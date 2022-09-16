<!-- master#1363d78 --->

# 教程：calc 语言

{{#include caveat.md}}

> 译者注：这一章的完整代码在 [calc-example] 下。

[calc-example]: https://github.com/salsa-rs/salsa/tree/master/calc-example/calc 

本教程将介绍一个使用 Salsa 的完整示例。本文并不假定你对 Salsa 有任何了解，但先阅读[概述][overview]能让你熟悉基本的概念。

[overview]: ./overview.md

我们的目标是给名为 `calc` 的简单语言定义一个编译器/解释器。 `calc` 编译器获取如下所示的程序，然后解析并执行它们：

```
fn area_rectangle(w, h) = w * h
fn area_circle(r) = 3.14 * r * r
print area_rectangle(3, 4)
print area_circle(1)
print 11 * 2
```

执行时，该程序打印 `12`、 `3.14` 和 `22`。

如果程序有错误（例如，使用未定义的函数），它也会打印出来。

当然，它将是反应式的，因此对输入的微小更改不需要重新编译（或重新执行）整个程序。
