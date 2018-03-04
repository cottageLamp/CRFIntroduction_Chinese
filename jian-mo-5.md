#2.5特征工程

<font color=red>不知道怎么翻译这里的专业名词</font>

这一节，我们讲述一些特征工程中的技巧。虽然主要用于语言处理，它们还是很通用的。最主要的权衡很典型——大的特征集可以提高预测的精度，因为决策便捷更加灵活，但却需要更大的内存来保存参数，且可能因为过拟合而降低预测精度。

**标签-观测特征?Label-observation features**.首先，当标签是离散变量，那么团模板$$\mathcal{C}_p$$的特征$$f_{pk}$$常常采用如下的特定形式：

$$
f_{pk}(y_c,x_c)={1}_{\{y_c=\tilde{y}_c\}}q_{pk}(x_c) (2.28)
$$

也就是说，一个特征只在输出正好为$${\tilde{y}}_c$$时才非零，而一旦如此，便只与输入有关。 我们把具有这种形式的特征称为标签-观测特征。本质上可以这么来理解：特征只依赖于输入$${x}_c$$，但每一种输出都有自己的一组权重。这一特征表示法的计算效率也很高，因为计算每个$$q_{pk}$$都可能涉及文本或图片处理，而只需要处理一次，就可用于每一个用到它的特征。为了避免混淆，我们把函数$$q_{pk}({x}_c)$$叫做观测函数，而不是特征。观测函数的例子有“单词$$x_t$$是大写的”或“单词$$x_t$$以ing结尾”。

**Unsupported Features.**使用标签-观测特征可能会带来数量庞大的参数。例如在CRFs的第一个大规模应用中，Sha和Pereira【125】在他们的最佳模型中，使用了3百8十万个参数。其中的很多特城从未在训练数据中出现过——它们总是0。原因在于，许多观测函数只与一小部分的标签相对应。例如在命名实体识别任务中，“单词$$x_t$$是with，而标签$$y_t$$是CITY-NAME”，似乎永远不可能在训练集中为真。我们把它们称为unsupported features。可能很意外，这些特征也可能有用，因为可以给它们赋予负的权重，从而防止给错的标签以高的概率。(降低那些从未出现过的标签序列的分数，将会增加那些出现过的标签序列的概率，所以在后文我们描述的参数估计方法中，会给这些特征以负的权重)。包含unsupported features常常带来精度的少量提升，并以巨大的参数数量为代价。

我们曾利用一个特别的技术，来选择unsupported features的一小部分。这可以看成是使用更少内存来利用的unsupported feature的一次简单探索，可以被称为“unsuported features trick”。它认为许多unsupported features是无用的，因为模型不太可能因为它们的激活而犯错。例如，那个“with”特征不太可能有用，因为with是一个常见的单词，且总是属于OTHER标签(即它不是一个名词)。为了减少参数的数量，我们只保留那些有可能剔除错误的unsupported features。一个简单的方法是：首先训练一个不带unsupported feature的CRF，并在几次迭代后就停下来，使得模型并没有完全训练好。然后考虑那些模型未能给正确答案以高概率的团，给它们增加unsupported features。在上面这个例子中，如果我们发现训练集中有一个样本i，其t位置的序列$$x_t^{(i)}$$是with，而$$y_t^{(i)}$$不是“CITY-NAME”<font color=red>原文是$$y_t^{(i)}$$ is not CITY-NAME。我认为应去掉not。译文则保留了这个not</font>，并且$$p(y_t=CITY-NAME |{x}_T^{(i)})>\epsilon$$时($$\epsilon$$时一个阈值)，我们增加"with"这一特征。

**连线-观测特征和节点-观测特征?Edge-Observation and Node-Observation Features.**为了减少模型中的特征数量，我们可以只在某些团使用标签-观测特征，而不是全部。最常见的两种标签-观测特征是*连线-观测特征*和*节点-观测特征*。考虑一个具有M个观测函数$$\{q_m({x})\}, m\in\{1,2,\cdots,M\}$$的线性链CRF。如果使用了连线-观测特征，那么每个局部函数可以依赖于全部的观测函数。那么，我们可以使用这样的特征：单词$$x_t$$是New，$$y_t$$是LOCATION 且$$y_{t-1}$$也是LOCATION。这会导致模型拥有大量的参数，带来内存消耗和过拟合的缺点。一种解决办法是采用节点-观测特征。使用这一类型的特征，转移因子<font color=red>就是局部函数吧?</font>不在依赖于观测函数。于是我们可以使用类似“$$y_t$$是LOCATION，且$$y_{t-1}$$是LOCATION”，以及“$$x_t$$是NEW，且$$y_t$$是LOCATION”的特征，而不能使用那种一次把$$x_t,y_t,y_{t-1}$$都依赖上的特征。连线-观测特征和节点特征都正式地在表2.1中给出了。一般来说，以上两种特征的选择，需要根据具体的问题来定，如需要考虑观测函数的数量，以及数据集的大小。

![](/assets/table 2.1.png)

**Boundary Labels.**最后一个问题是如何在边缘上取标签，例如一个序列的开始和结尾，或一张画的边缘。有时，边缘上的标签与其他标签不同。例如，大写字母在一个句子的中间意味着是专有名词，但如果是在句子的开始却没有这样的意味。一个简单的办法，是在标签序列的前面加一个特殊的标签——START。这允许模型学习得到关于边缘的特性。例如，如果连线-观测特征也被使用了，那么像“$$y_{t-1}=START$$且$$y_t=PERSON$$且$$x_t$$大写”这样的特征，可以表示，大写这一特征在句子的开始时并不是有效的。

**特征归纳？Feature Induction**上文介绍的“unsupported features trick”是“feature induction”的简化版。McCallum【83】提供了CRFsf 特征归纳的更有条理的方法。其中，模型一开始只有一些基本特征，而训练过程会增加这些特征的连接。另外一个选择是特征选择。一个现代的特征选择方法是$$L_1$$规则化。我们将在第5.1.1介绍它。Lavergne et al.[65]发现，在最好的时候，$$L_1$$可以找到一种模型。它只有1%的参数是非零的，却获得与稠密特征集相当的性能。他们还发现，利用$$L_2$$规则化目标函数，来对$$L_1$$规则化所得的非零特征进行微调，也是有用的。

**Categorical Features类属特征(非数值特征).**如果观测是类属的，而不是有序的，就是说，它们是离散而没有内在的顺序性，那么将它们转化成二值化特征是重要的。例如，很合理将特征$$f_k(y,x_t)$$定义为“如果$$x_t$$是单词dog时，$$f_k=1$$，否则为0”。相反，把$$f_k$$定义为单词$$x_t$$在文本词典中的序号是不合理的。 因而在文本处理中，CRF特征常常是二值化的；而在其他诸如视觉和语音识别中，特征常常是数值的。对于数值特征，标准的做法是通过归一化，使其均值为0而标准差为1，或者把它们二值化，使其变成类属特征。

**Features from Different Time Steps.**我们对于特征$$f_k(y_t,y_{t-1},{x}_t)$$的关注可能遮掩了一点，即通常需要让特征的依赖范围，从最近邻扩展到附近的标签。一个这种特征的例子是“单词$$x_{t+2}$$是Times，而标签$$y_t$$是ORGANIZATION“。这有利于识别名词”New York Times"报纸。同样，也临近特征的组合也是有用的，例如“单词$$x_{t+1}$$和$$x_{t+2}$$是York Times”。

**Features as Backoff回退特征？.**在语言处理中，有时需要在模型中包含冗余因子。例如在线性链CRF中，有人会使用连接因子$$\Psi_t(y_t,y_{t-1},{x}_t)$$的同时，还使用变量因子$$\Psi_t(y_t,{x}_t)$$。虽然只使用连接因子也可以定义同样的分布簇，然而当数据量小于特征的数量时，冗余节点因子却像回退语言模型那样有用。(当拥有百万级的特征时，很多数据是很小的！)当使用冗余特征时，规则化(5.1.1节)是很必要的，因为惩罚大的权重会让权重分布到重叠的特征上。

**Features as Model Combination.**另一种有意思的特征可以是相同任务的更简单方法的结果。例如，如果已经拥有了任务的简单规则库simpl'e rule-base系统(例如这样的规则“1900和2100中间的数字字符串表示一个年份)，那么该系统的输出可被用做CRF的观测函数。另一个例子是名录特征gazetteer features，即其观测函数建立在一个预先建立的列表上，如”如果$$x_t$$出现在了Wikipedia提供的某个城市名单列表中，那么$$q(x_t)=1$$“。

更复杂的例子是把生成模型的输出当做判别模型的输出来用。例如人们可以使用$$f_t(y,{x}_t)=p_{HMM}(y_t=y|x)$$作为特征，其中$$p_{HMM}$$表示某个HMM(在相近数据集训练所得的)所给出的$$y_t=y$$的边缘概率。 让HMM和CRF-with-HMM-feature在同一个数据上训练通常不是一个好的想法，因为HMM需要在它自己的数据集上表现极好，而这会让CRF过分依赖与HMM。这一技术可用于提高某个早前的、同一任务的系统的性能。Bernal et al【7】是这一概念的、在DNA序列中识别基因的一个好例子。

相关的想法是对输入$${x}_t$$进行聚类，用任何方法对语料库中的单词进行聚类，然后用类别标签来作为单词$$x_t$$的附加特征。这种特征在Miller et al.[90]那里取得了好的效果。

**Input-Dependent Structure.**在通用CRF中，有时需要让$$p({y|x})$$的图结构随着输入x变化。关于此的一个简单例子是关于文本处理的“skip-chain CRF”【37,117,133】。其背后的思想是，一旦某个单词在句子中出现了两次，我们希望它们属于相同的标签。于是我们在这两个单词中间增加一条连接特征。这让y之上的图结构依赖于输入x。