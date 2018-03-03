#2.4 通用CRFs

现在，我们将刚刚探讨的线性链扩展到通用图，以与Lafferty在【63】中对CFR的定义相匹配。概念上，这一扩展是显而易见的。我们只需简单地把线性链因子图变成通用因子图。

--------------

**定义2.3** 记G是在X,Y上的因子图。如果对于X中任意的值x，分布p(y|x)是根据G来分解的，那么$$(X,Y)$$是一个**条件随机场conditional random field**。

--------

那么，每个条件分布p(y|x)都是某些因子图的CRF，包括是平凡的。如果$$F\in\{\Psi_a\}$$是G中的因子的集合，那么一个CRF的条件分布为：

$$
p(y|x)=\frac{1}{Z(X)}\prod^A_{a=1}\Psi_a(y_a,x_a) (2.22)
$$

本定义相比一般无向图的定义(2.1)，差别在于归一化常数Z(x)现在变成了关于输入x的函数。因为条件性趋向于简化图模型，Z(x)有可能被计算，而Z却不是。

正如我们在HMMs和线性链CRFs中的做法，让$$\Psi_a$$是一组特征的线性函数是有用的，即：

$$
\Psi_a(y_a,x_a)=exp\left\{\sum^{K(A)}_{k=1}\theta_{ak}f_{ak}(y_a,x_a)\right\} (2.23)
$$

其中特征函数$$f_{ak}$$和权重$$\theta_{ak}$$都使用了因子的下标a，这是为了强调每个因子都有自己的权重集。一般来说，每个因子也可以拥有自己的特征函数。注意，如果x和y是离散的，那么(2.23)中的log-线性假设并没有带来额外的局限，因为我们可以给(y_a,x_a)的每一个值安排一个指示函数$$f_{ak}$$，类似于我们把HMMs转变成线性链CRF时的做法。

综合(2.22)和(2.23)，可以把log-线性因子CRF的条件分布写成

$$
p(y|x)=\frac{1}{Z(x)}\prod_{\Psi_A\in F}exp\left\{\sum^{K(A)}_{k=1}\theta_{ak}f_{ak}(y_a,x_a)\right\} (2.24)
$$

另外，许多应用模型常常需要参数绑定。以线性链为例，每一时刻的因子$$\Psi_t(y_t,y_{t-1},{x}_t)$$常常使用相同的权重。为了表示这一情况，我们把G的因子划分成$$\mathcal{C}=\{C_1,C_2,\cdots,C_P\}$$，其中每个$$C_P$$是一个**团模板clique template**，是一组共享了特征函数$$\{f_{pk}(x_c,y_c)\}^{K(p)}_{k=1}$$和参数$$\theta_p\in \mathcal{R}^{K(p)}$$的因子。一个使用了团模板的CRF可以写成

$$
p(y|x)=\frac{1}{Z(x)}\prod_{C_p\in\mathcal{C}}\prod_{\Psi_c\in C_p}\Psi_c(x_c,y_c;\theta_p) (2.27)
$$

其中每个模板因子是这样参数化的

$$
\Psi_c(x_c,y_c;\theta_p)=exp\left\{\sum^{K(p)}_{k=1}\theta_{pk}f_{pk}(x_c,y_c\right\} (2.26)
$$

而归一化函数为

$$
Z(x)=\sum_y\prod_{C_p\in\mathcal{C}}\prod_{\Psi_c\in C_p}\Psi_c(x_c,y_c). (2.27)
$$

这一团模板的记号方法即指明了结构重复，也指明了参数绑定。以线性链CRF为例，典型的团模板$$C_0=\{\Psi_t(y_t,y_{t-1},{x}_t)\}^T_{t=1}$$倍整个网络使用，因而$$\mathcal{C}=\{C_0\}$$是元素单一的集合。如果相反地，我们希望给每个因子$$\Psi_t$$分配独立的参数，就像非齐次HMM，那么需要T个模板，即$$\mathcal{C}=\{C_t\}^T_{t=1}, C_t=\{\Psi_t(y_t,y_{t-1},{x}_t)\}$$。

定义通用CRF时，如何给出重复的结构以及参数绑定，是属于最需要考虑的问题。人们推荐了一系列的规范，用于指定团模板，而我们仅仅在这里简单的罗列一下。例如，**动态条件随机场dynamic conditional random field**[140]是一些序列模型，允许在每个时刻拥有多个标签<font color=red>译注：不是指有多个类别，而是有多个变量</font>，而不只是单一的标签，很像动态贝叶斯网络。第二，**关系马尔科夫网relational Markov networks**【142】，是一种用类SQL的语法来指明图结构和参数绑定的通用CRF。**马尔科夫逻辑网Markov logic networks**【113,128】用逻辑式子(logic formulae)来给出无向图的局部函数的分数。实质上，知识库中的每条一阶规则都存在一组参数。MLN的逻辑部分，本质上，可以被看成一种编码惯例，用来指明无向图中的重复结构以及参数绑定。Imperatively define factor graphs【87】使用了完整表达的Turing-complete函数来定义团模板，即给出了模型的结构，也给出了充分统计量$$f_{pk}$$。这些函数灵活地采用了先进的编程思想，包括递归、任意搜索(arbitrary search)、惰性计算以及记忆化。本文采用的团模板的记号，来自于Taskar et al.[142]， Sutton et al. [140]，Richardson 和 Domingos [113]，以及McCallum et al.[87]
