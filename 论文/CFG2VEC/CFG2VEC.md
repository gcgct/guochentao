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
* 我们建议在训练时使用跨体系结构损失，允许cfg2vec捕获二进制文件的体系结构不可知表示。
* 我们在GitHub存储库中发布cfg2vec: https://github.com/AICPS/mindight/cfg2vec。
* 我们将我们的cfg2vec集成到实验Ghidra插件中，协助修补DARPAAssured MicroPatching (AMP)挑战二进制文件的现实场景

​	如果我们有一个为一种架构编译的剥离二进制文件和一个为另一个带有调试符号的架构编译的可比程序的二进制，则执行跨架构函数名称预测/匹配的工具将是有益的。我们可以使用带有调试符号的二进制来预测剥离二进制文件中函数的名称，这显着有助于调试。

### 模型

![image-20240224173411222](C:\Users\10231\AppData\Roaming\Typora\typora-user-images\image-20240224173411222.png)

给定一个二进制代码，表示为 p，由同的 CPU 架构编译，我们提取图 (GoG) 表示，G = (V , A)，其中 V 是节点集，A 是邻接矩阵（如图 3 所示）。V 中的节点表示函数，A 中的边表示它们的交叉引用关系。也就是说，每个节点 fi ∈ V 是一个 CFG，我们将其表示为 fi = (B, A, φ)，其中 B 中的节点表示基本块，A 中的边表示它们的依赖关系。φ 是一个映射函数，它将汇编形式中的每个基本块映射到其对应的提取属性 φ(vi) = Ck，其中 C 是数值，k 是基本块 (BB) 的属性数。虽然 CFG 结构旨在在较低的 BB 级别提供更多的信息，但 GoG 结构旨在在 CFG 之间的总体功能级别恢复信息。图 3 是部分 GoG 结构的示例，仔细检查其 CFG 节点之一而不是单个 CFG BB 节点之一，显示了与该 BB 节点对应的特征集。目标是设计一种高效且有效的图嵌入技术，可用于重建每个函数 fi ∈ V 的函数名称。

为了提取cfg2vec所需的结构化表示，我们利用最先进的反编译器Ghidra和Ghidra Headless Analyzer1。无头分析器是Ghidra的命令行版本，允许用户通过命令行界面执行Ghidra支持的许多任务(如分析二进制文件)。

为了从二进制文件中提取GoG，我们开发了 Ghidra Data Toolkit (GDT); GDT 是一组基于 Java 的元数据提取脚本，用于检测 Ghidra Headless Analyzer。首先，GDT 程序性地分析给定的可执行文件并将提取的信息存储在内部 Ghidra 数据库中。Ghidra 提供了一组 API 来访问数据库并检索有关分析二进制文件的信息。GDT使用这些 API 导出信息，例如 Ghirdda 的 PCode 和每个函数的调用图。具体来说，FunctionManager API 允许我们操纵二进制中每个反编译函数的信息并获得函数之间的交叉调用依赖关系。对于每个函数，我们利用另一个 Ghidra API DecompInterface2 来提取与函数中的每个基本块相关联的 12 个属性。这些属性精确地对应于指令的总数，包括算术、逻辑、传输、调用、数据传输、SSA、比较和指针指令，以及不属于这些类别的其他指令以及该 BB 中的常量和字符串的总数。最后，通过整合所有信息，我们为每个二进制 p 形成一个 GoG 表示 G。我们重复这个过程，直到所有二进制文件都转换为 GoG 结构。我们将生成的 GoG 表示分批提供给我们的模型，批量大小为 B。

一旦从 GDT 中提取 G，我们将其馈送到我们的分层网络架构，其中包含 CFG Graph Embedding 层和 GoG Graph Embedding 层，如图 4 所示。对于每个 GoG 结构，我们将其表示为 G = (V , A)，其中 V 是一组与 G 相关的函数，A 表示 V 中函数之间的调用关系。V 中的每个函数都以 CFG fi = (B, A, φ) 的形式，其中每个节点 b ∈ B 是一个以固定长度属性向量 b ∈ Rd 表示的 BB，并dis 是我们之前提到的维度。A 对这些 BB 之间的成对控制流依赖关系进行编码。



1.CFG图嵌入层：我们的网络架构首先将一批GoG中的所有函数馈送到由多个图卷积层和一个图读出操作组成的CFG图嵌入式层。该层的输入是函数fi=（B，a，φ），输出是表示函数的固定维向量。对于每个BBbk，我们让b0k=bk，并使用如下所示的图卷积运算将BTK更新为bt+1k

![image-20240224185801009](C:\Users\10231\AppData\Roaming\Typora\typora-user-images\image-20240224185801009.png)

其中 fG 是一个非线性激活函数，例如 ReLU，Ak 是 bk 的相邻 BB 列表，W ∈ Rd×d 和 M ∈ Rd×d 是训练期间要学习的权重。我们运行这种卷积的 T 次迭代，这可能是我们模型中的可调超参数。在更新期间，每个 BB 利用其邻居的表示逐渐将控制流依赖关系的全局信息聚合到其表示中。我们将每个 BB 的最终表示作为 bT k。为了获得函数fi的表示，我们应用了一个图读出操作，如sum-readout，描述如下:

![image-20240224185834619](C:\Users\10231\AppData\Roaming\Typora\typora-user-images\image-20240224185834619.png)

我们分配 g(T ) 的值（又名CFG 嵌入）到 fi。图读出操作可以用mean-readout或max-readout代替。

2.GoG 图嵌入层：一旦所有函数都转换为固定长度的图嵌入，我们将 G 馈送到 cfg2vec 的第二层，即 GoG 嵌入层。在这里，对于每个函数 fi，我们应用另一个带有 F 和 C 的图卷积迭代。更新可以表示如下，

![image-20240224185918560](C:\Users\10231\AppData\Roaming\Typora\typora-user-images\image-20240224185918560.png)

其中 fGoG 是一个非线性激活函数，Ck 是函数 fk 的相邻函数（调用）列表，U ∈Rd×d 和 V ∈ Rd×d 是训练期间要学习的权重。最后，我们将 f (L)k 作为同时考虑 CFG 和 GoG 图结构的表示。我们使用这些更新的表示来执行跨架构函数相似性学习。

![image-20240224185953691](C:\Users\10231\AppData\Roaming\Typora\typora-user-images\image-20240224185953691.png)

基于连体的交叉架构函数相似度：给定一批 GoG B = {GoG1, GoG2,。.., GoGB }，我们应用分层图神经网络来获取更新的函数嵌入集，表示为 BF ={f (T ) 1 , f (T ) 2 ,。.., f (T )K }。我们计算具有余弦相似度的每个函数对的函数相似度，表示为 ^y ∈ [−1, 1]。^Y 和真实标签 y 之间的损失函数 J，它表示一对函数是否具有相同的函数，计算如下：

![image-20240224190030590](C:\Users\10231\AppData\Roaming\Typora\typora-user-images\image-20240224190030590.png)

然后将最终损失 L 计算如下

![image-20240224190050316](C:\Users\10231\AppData\Roaming\Typora\typora-user-images\image-20240224190050316.png)

其中 Y 代表真实标签（相似性或差异），^Y 代表相应的预测。更具体地说，如果它们相同，但使用不同的 CPU 架构编译，我们将一对函数表示为相似的。m 是一个常数，以防止学习到的嵌入变得扭曲（默认情况下，0.5）。为了保持正负训练样本之间的平衡，我们开发了一种自定义批处理算法。该函数利用通过向给定批次添加某个包的二进制来查找并添加为不同架构构建的相同包的二进制获得的知识，将提供的批次作为正样本。它还将包含来自另一个包的二进制作为负样本。这将给任何批次提供平衡比例的正负样本。最后，我们使用损失 L 使用 Adam 优化器更新神经网络中的所有相关权重。一旦经过训练，我们使用该模型来执行函数名称重建任务。





