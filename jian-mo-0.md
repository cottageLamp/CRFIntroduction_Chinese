# 2 建模

本章，我们从建模的角度来描述CRFs，阐述了CRF是如何把机构化的输出表示成高维输入向量的分布。可以把CRFs理解成，将逻辑回归分类器扩展到任意的图模型，也可以被理解成生成模型(如隐马尔科夫模型)的判别对应物。<font color=red>译注：**判别**和**生成**模型是两种在理论上等价(可互相推导得到对方)，但建模思路相反的模型</font>。

我们从对图模型的简单介绍(第2.1节)，以及对NLP中的生成和判别模型的介绍(第2.2节)开始。然后，我们可以给出了CRF的正式定义，包括常用的线性链(linear chains)(第2.3节)，以及通用图结构(第2.4节)。因为CRF的准确性严重依赖于所使用的特征，我们也描述了特征工程常用的一些技巧(第2.5节)。最后，我们提供两个CRF应用的例子(第2.6节)，以及一个宽泛的、关于CRFs应用领域的报告。