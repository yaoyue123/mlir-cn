# Toy 入门教程

本教程演示了如何在MLIR之上实现一个基本的toy语言。该教程的目标是介绍MLIR的概念，特别是如何使用[dialects](../../LangRef.md/#dialects)来轻松支持特定于语言的结构和转换，同时仍然提供一个方便的路径到LLVM或其他codegen基础设施。本教程基于[LLVM Kaleidoscope Tutorial教程](https://llvm.org/docs/tutorial/MyFirstLanguageFrontend/index.html)..

另一个很好的介绍来源是网上[视频](https://www.youtube.com/watch?v=Y4SvqTtOIDk)
来自2020年LLVM开发大会([幻灯片](https://llvm.org/devmtg/2020-09/slides/MLIR_Tutorial.pdf)).

本教程假设您已经克隆并构建了MLIR；如果您还没有这样做，请参阅以下链接以获取有关如何构建MLIR的说明
[开始使用MLIR](../../../_index.md).

本教程分为以下章节:

-   [第1章](Ch-1.md): 介绍Toy语言及其AST的定义。
-   [第2章](Ch-2.md): 遍历AST以在MLIR中发出一个dialect，介绍基本的MLIR概念。在这里，我们将展示如何开始在MLIR中为我们的自定义操作附加语义。
-   [第3章](Ch-3.md): 使用模式重写系统进行高级语言特定的优化。
-   [第4章](Ch-4.md): 使用接口编写通用的dialect无关变换。在这里，我们将展示如何将dialect中特定的信息插入到通用变换中，如形状推断和内联。
-   [第5章](Ch-5.md): 部分转换到较低级别的方言。我们将把一些高级语言特定的语义转换为一个面向优化的通用dialect方言。
-   [第6章](Ch-6.md): 底层到LLVM和代码生成。在这里，我们将以LLVM IR为目标进行代码生成，并详细介绍底层框架。
-   [第7章](Ch-7.md): 扩展Toy：添加对复合类型的支持。我们将演示如何向MLIR添加自定义类型以及它如何适应现有的流水线。

[第一章](Ch-1.md) 将介绍Toy语言和AST。