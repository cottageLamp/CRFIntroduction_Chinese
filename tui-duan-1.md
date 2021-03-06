# 4.1 线性链CRFs

这一节，我们简要介绍HMMs的标准推断算法——前向后向以及Viterbi算法，以及如何将它们应用在线性链CRFs上。Rabiner的【111】是一份关于这些算法在HMM上的研究。所有这些算法都只是第4.2.2节将要描述的置信传播算法的特例。然而，我们仍将详细讨论这一在线性链上的特例，因为它能让后面的讨论更具体，也因为它自身在工程上就很有用。

首先，我们引入一些记号，能简化接下来的前向后向递归(forward backward recursion)。一个HMM可以写成Z=1的因子图$$p(y,x)=\prod_t\Psi_t(y_t,y_{t-1},x_t)$$，而因子被定义为
$$
\Psi_t(j,t,x)\stackrel{def}{=}p(y_t=j|y_{t-1}=i)p(x_t=x|y_t=j) (4.1)
$$
如果把这个HMM看成带权重的有限状态机，那么$$\Psi_t(j,i,x)$$就是当观测为x时，从状态i变成j的权重。

现在我们来研究HMM的前向算法，这是用来计算观测值的概率p(x)的。前向后向算法背后的思想是，首先把$$p(x)=\sum_yp(x,y)$$的求和运算，按照如下的方式重写

$$
p(x)=\sum_y\prod^T_{t=1}\Psi_t(y_t,y_{t-1},x_t)
$$

$$
=\sum_{y_T}\sum_{y_{T-1}}\Psi_T(y_T,y_{T-1},x_T)\sum_{y_{T-2}}\Psi_{T-1}(Y_{T-1},y_{T-2},x_{T-1})\sum_{y_{T-2}}\cdots(4.3)
$$

现在我们可以看到，在进行外部的求和运算时，其内部的求和运算要被反复调用。因此，我们可以把里面的保存起来，从而爆炸式地减少了计算量。

这导致了所谓的前向变量$$\alpha_t$$，是大小为M的向量(M是状态的数量)，用来保存求和的中间结果。其定义为：

$$
\alpha_t(j)\stackrel{def}{=}p(x_{<1\cdots t>},y_t=j)
$$

$$
=\sum_{y_{<1\cdots t-1>}}\Psi_t(j,y_{t-1},x_t)\prod^{t-1}_{t'=1}\Psi_{t'}(y_{t'},y_{t'-1},x_{t'}) (4.5)
$$

其中，求和运算的下标$${y}_{1\cdots t-1}$$，表示要覆盖$$y_1,y_2,\cdots,y_{t-1}$$的所有可能值。这一alpha可以通过递归的方式计算

$$
\alpha_t(j)=\sum_{i\in S}\Psi_t(j,i,x_t)\alpha_{t-1}(i) (4.6)
$$

而初始变量为$$\alpha_1(j)=\Psi_1(j,y_0,x_1)$$。(回忆(2.10)，知道$$y_0$$是HMM的固定的初始值)。反复地递归(4.6)式，可知$$p(x)=\sum_{y_T}\alpha_T(y_T)$$。正式地证明应该需要数学归纳法。

后向递归于此相同，除了在(4.3)中把求和的顺序颠倒过来。所得的定义为

$$
\beta_t(i)\stackrel{def}{=}p(x_{<t+1\cdots T>}|y_t=i)
$$

$$
=\sum_{y_{<t+1\cdots T>}}\prod^T_{t'=t+1}\Psi_{t'}(y_{t'},y_{t'-1},x_{t'}) (4.8)
$$

而其递归为

$$
\beta_t(i)=\sum_{j\in S}\Psi_{t+1}(j,i,x_{t+1})\beta_{t+1}(j) (4.9)
$$

其中，初始量为$$\beta_T(i)=1$$。与前向类似，我们也可以用后向变量来计算$$p(x)=\beta_0(y_0)\stackrel{def}{=}\sum_{y_1}\Psi_1(y_1,y_0,x_1)\beta_1(y_1)$$。

要计算边缘分布$$p(y_{t-1},y_t|x)$$(在参数估计时需要)，我们要把前向和后向的结果综合起来。这可以从概率或因子分解的角度来看。首先，从概率的角度来看，我们写成

$$
p(y_{t-1},y_t|x)=\frac{p(x|y_{t-1},y_t)p(y_{t-1},y_t)}{p(x)}
$$

$$
=\frac{p(x_{<1\cdots t-1>},y_{t-1})p(y_t|y_{t-1})p(x_t|y_t)p(x_{<t+1\cdots T}|y_t)}{p(x)}
$$

$$
=\frac{1}{p(x)}\alpha_{t-1}\Psi_t(y_t,y_{t-1},x_t)\beta_t(y_t) (4.12)
$$

在上式的第二行中，我们基于如下的事实：给定$$y_t,y_{t-1}$$后，$$x_{<1\cdots t-1>}$$与$$x_{<t+1\cdots T>}$$以及$$x_t$$无关。同样的，从因子分解的角度看，我们利用分配率得

$$
p(y_{t-1},y_t|x)=\frac{1}{p(x)}\Psi_t(y_t,y_{t-1},x_t)
$$

$$
\times \left(\sum_{y_{<1\cdots t-2>}}\prod^{t-1}_{t'=1}\Psi_{t'}(y_{t'},y_{t'-1},x_{t'})\right)
$$

$$
\times \left(\sum_{y_{<t+1\cdots T>}}\prod^{T}_{t'=t+1}\Psi_{t'}(y_{t'},y_{t'-1},x_{t'})\right)
$$

然后，通过带入$$\alpha$$与$$\beta$$的定义，我们得到与前文一样的结果，即：

$$
p(y_{t-1},y_t|x)=\frac{1}{p(x)}\alpha_{t-1}(y_{t-1})\Psi_t(y_t,y_{t-1},x_t)\beta_t(y_t) (4.14)
$$

而$$1/p(x)$$相当于分布的归一化常数。我们通过$$p(x)=\beta_0(y_0)$$或$$p(x)=\sum_{i\in S}\alpha_T(i)$$来计算它。

总的来说，前向后向算法就是：首先用(4.6)计算每个$$\alpha_t$$，然后用(4.9)计算每个$$\beta_t$$，然后用(4.14)计算边缘分布。

最后，如要计算最可能的输出$$y^*=\arg\max_{y}p(y|x)$$，我们发现之前在(4.3)中使用的技巧仍然有效。这带来了Viterbi算法。与前向变量$$\alpha$$像类似的变量为

$$
\delta_t(j)\stackrel{def}{=}\max_{y_{<1\cdots t-1>}}\Psi_t(j,y_{t-1},x_t)\prod^{t-1}_{t'=1}\Psi_{t'}(y_{t'},y_{t'-1},x_{t'}) (4.15)
$$

而这可以通过类似的递归来计算:

$$
\delta_t(j)=\max_{i\in S}\Psi_t(j,i,x_t)\delta_{t-1}(i) (4.16)
$$

$$\delta$$被计算出来之后，最可能的输出可通过如下的后向递归计算:

$$
y_T^*=\arg\max_{i\in S}\delta_T(i)
$$

$$
y_t^*=\arg\max_{i\in S}\Psi_t(y^*_{t+1},i,x_{t+1})\delta_t(i)\ for\ t<T
$$

对$$\delta_t$$和$$y^*_t$$的递归，构成了**Viterbi算法**。

现在，我们已经讲述了HMMs的前向后向和Viterbi算法。将其扩展到线性链CRFs是直接的。线性链CRFs的前向后向算法与HMMs的一样，只是转移权重$$\Psi_t(j,i,x_t)$$的定义需要改变。我们注意到，(2.18)的CRF模型可以重写为

$$
p(y|x)=\frac{1}{Z(x)}\prod^T_{t=1}\Psi_t(y_t,y_{t-1},x_t),(4.17)
$$

其中

$$
\Psi_t(y_t,y_{t-1},x_t)=exp\left\{\sum_k\theta_k f_k(y_t,y_{t-1},x_t)\right\} (4.18)
$$

使用这里的定义，前向递归(4.6)、后向递归(4.9)以及Viterbi递归(4.16)可不经过修改就用于线性链CRFs，只是其含义有所改变。在CRF中，$$\alpha_t(j)=p(x_{<1\cdots t>},y_t=j)$$不再具有概率的含义，而需从因子分解的角度来理解。就是说，我们需根据(4.5)定义$$\alpha$$，(4.8)定义$$\beta$$，(4.15)定义$$\delta$$。同样地，前向后向递归的结果现在变成了Z(x)，而不是p(x)，且$$Z(x)=\beta_0(y_0),Z(x)=\sum_{i\in S}\alpha_T(i)$$。

关于边缘分布，方程(4.14)仍然有效，只是需用Z(x)替换p(x)，即

$$
p(y_{t-1},y_t|x)=\frac{1}{Z(x)}\alpha_{t-1}(y_{t-1})\Psi_t(y_t,y_{t-1},x_t)\beta_t(y_t) (4.19)
$$

$$
p(y_t|x)=\frac{1}{Z(x)}\alpha_t(y_t)\beta_t(y_t) (4.20)
$$

我们补充三个可被上面的算法直接解的推断任务。一，如果我们想从后验概p(y|x)中采样y，可使用前向算法+后向采样过程，就如在HMMs中的做法一样。二，如果不想只找到唯一的最可能输出$$\arg\max_{y}p(y|x)$$，而是前k个可能输出，我们可以使用HMMs的标准算法[129]。最后，有时候我们要计算一组节点$$S\subset[1,2,\cdots,T]$$(不一定是连接在一起的)的边缘概率$$p({y_S|x})$$。例如，这可用于评估模型在部分输入上的性能。这一边缘概率可使用Culotta和McCallum【30】描述的带约束前向后向算法来解。