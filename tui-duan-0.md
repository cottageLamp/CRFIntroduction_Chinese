# 4 推断

推断的效率对CRFs至关重要，无论是对训练还是预测。有两个关于推断的任务。一是给定新的输入x后，要预测最可能的输出$$y^*=\arg\max_{y}p(y|x)$$。二是，正如第5节所述，参数估计时所需的边缘分布，如单个节点的$$p(y_t|x)$$和连接的$$p(y_t,y_{t-1}|x)$$。这两个任务可被看成two different smirings下的同一操作。就是说，把边缘概率问题改成求最大值问题，我们只需简单地把求和运算变成求最大运算。

对于离散的情况，可通过穷举的办法计算边缘概率，然而所需的计算时间会因Y的尺寸而指数爆炸。实际上，对于通用图来说，关于推断的两个问题都是困难的，因为任何*命题可满足性问题propositional satisfiability problem*都可以轻易地用因子图来表示。

可快速而精确地解线性链CRFs，其方法是HMMs的动态规划算法的变体。在第4.1节，我们从计算边缘分布的forward-backward算法以及计算最可能赋值的Viterbi算法开始。这些算法那是通用的置信传播算法belief propagation algorithm在树图模型(4.2.2)上的特例。对于更复杂的模型，需要近似的推断算法。

可以说，CRF的推断问题与一般的图模型并无差别，因而一般图模型的推断算法也都适用，如一些教材【57,59】所言。然而关于CRFs，我们需要时刻注意两点。第一，在参数估计(5.1.1)时需要反复执行推断任务，非常耗时，因而我们希望能在计算效率和准确性之间做些权衡。第二，若采用了近似推断，那可能会带来推断过程与训练过程之间复杂的相互作用。我们把这一议题延后至第5节，因为我们将在那里讨论参数估计，然而有必要在这里提出这个问题，因为它严重影响着对推断算法的选择。