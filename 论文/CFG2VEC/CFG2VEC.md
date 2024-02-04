## CFG2VEC: Hierarchical Graph Neural Network for Cross-Architectural Software Reverse Engineering

### 概念

* Graph-of-Graph（`GoG`）是一种图数据结构，其中图的节点本身也是图。节点可以包含一系列的子图，这些子图可以是有向图或无向图

* GNN 是图神经网络（Graph Neural Network）的简称，它是一种专门用于处理图数据的机器学习模型。传统的神经网络模型主要适用于处理向量或矩阵形式的数据，而图神经网络则扩展到了更一般化的图结构数据。基本组成部分包括节点嵌入（node embedding）和图汇聚（graph pooling）两个主要步骤：

  1. 节点嵌入：节点嵌入是指将每个节点的特征映射到一个低维向量空间中，以便进行后续的计算和学习。常见的节点嵌入方法包括图卷积网络（Graph Convolutional Network，GCN）、GraphSAGE、Graph Attention Network（GAT）等。
  2. 图汇聚：图汇聚是指对整个图的信息进行聚合，以获得图级别的表示。在图神经网络中，图汇聚可以通过池化操作或者注意力机制来实现。常见的图汇聚方法包括图池化网络（Graph Pooling Network，GPN）、图注意力网络（Graph Attention Network，GAT）等

* `FCG`（Flow Control Graph）是一种用于表示程序控制流的图形结构。与`CFG`（Control Flow Graph）类似，但FCG是对CFG的扩展，包含了更多的信息，以提供更丰富的程序分析和优化支持。

* 图嵌入技术是一种将图形数据中的节点和边表示为低维向量的方法。它在图分析和图数据挖掘的任务中发挥着重要作用。

* 图嵌入 GE（Graph Embedding GE）是指使用图嵌入技术将图形数据中的节点或边表示为低维向量的过程。

* 图卷积层（Graph Convolutional Layer）是一种用于图神经网络（Graph Neural Networks，GNN）的核心组件。它通过在图结构上进行卷积操作来学习节点的表示。

* 图池化层（Graph Pooling Layer）是图神经网络（Graph Neural Networks，GNN）中的一种操作，用于减少图的规模和维度，并提取更高层次的图表示。

* PCode（Portable Code）是一种中间代码（Intermediate Code）表示形式，用于在不同的计算机体系结构和编程语言之间进行代码转换和移植。通常是一种类似于汇编语言的低级表示形式

* 连体网络（Siamese Network）是一种深度神经网络结构，用于学习和比较两个输入之间的相似性或差异性。它的名字来源于双胞胎中的连体兄弟或连体姐妹，因为它包含了两个共享参数的子网络，这些子网络在结构上是相同的，且参数是共享的。

  ![image-20240204194548452](C:\Users\10231\AppData\Roaming\Typora\typora-user-images\image-20240204194548452.png)

  

### 局限

编译过程丢弃了源级信息，并降低了其抽象级别，以换取更小的占用大小、更快的执行时间甚至安全考虑。评论、变量名称、函数名称和惯用结构等源级信息对于理解程序至关重要，但在这些反编译器的输出中通常不可用.通过分析符号名称、类型和位置来匹配函数名称。然而，只能从预定的封闭集进行预测，无法泛化到新名称。

### 贡献

* 我们提出了一种新的方法cfg2vec，它使用分层图神经网络(GNN)来建模二进制文件中的控制流和函数调用关系。
* 我们建议在训练时使用跨体系结构损失，允许cfg2vec捕获二进制文件的体系结构不可知表示。•我们在GitHub存储库中发布cfg2vec: https://github.com/AICPS/mindight/cfg2vec。
* 我们将我们的cfg2vec集成到实验Ghidra插件中，协助修补DARPAAssured MicroPatching (AMP)挑战二进制文件的现实场景

### 模型





