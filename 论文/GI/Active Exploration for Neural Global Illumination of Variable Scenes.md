# 摘要
提出了一种神经渲染方法，可以一边训练一边生成训练数据

# 方法
定义一个场景配置的解空间$\mathcal{D}$，也就是所有可能的场景配置。而一个特定的场景就是这个解空间中的一个向量$v\in\mathcal{D}$。本文的方法就是训练一个网络，输入一系列GBuffer和场景向量$v$，推理全局光照图像。
$\mathcal{D}$是一个及其高维的空间，为了更好地生成样本，引入了一个基于马尔科夫链的样本重用策略。
## 显式场景表示
为了避免使用直接的编码器对场景进行神经编码（缺乏可解释性），本文采用了显式的场景表达。具体来说，仅将那些可变的参数收集起来，作为场景的可变部分。比如cornell box的墙壁颜色、光源位置等，如下图：
![Explicit Scene Representation](18.png)
这么做的好处是，网络中将编码所有的场景静态部分(静态部分的信息由输入的GBuffer给出)，而场景的动态部分由这个向量给出。