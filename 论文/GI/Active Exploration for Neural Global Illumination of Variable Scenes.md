# 摘要
提出了一种神经渲染方法，可以一边训练一边生成训练数据

# 方法
## pipeline
整个pipeline如下：
![](20.png)
渲染部分：
输入场景描述和GBuffer，推理得到全局光照的结果。
训练部分：
定义一个场景配置的解空间$\mathcal{D}$，也就是所有可能的场景配置。而一个特定的场景就是这个解空间中的一个向量$v\in\mathcal{D}$。本文的方法就是训练一个网络，输入一系列GBuffer和场景向量$v$，推理全局光照图像。
$\mathcal{D}$是一个及其高维的空间，为了更好地生成样本，引入了一个基于马尔科夫链的样本重用策略。
## 显式场景表示
为了避免使用直接的编码器对场景进行神经编码（缺乏可解释性），本文采用了显式的场景表达。具体来说，仅将那些可变的参数收集起来，作为场景的可变部分。比如cornell box的墙壁颜色、光源位置等，如下图：
![Explicit Scene Representation](18.png)
这么做的好处是，网络中将编码所有的场景静态部分(静态部分的信息由输入的GBuffer给出)，而场景的动态部分由这个向量给出。
## 场景空间探索
所有合法的场景向量在进行规范化之后构成了一个高维立方体空间，每一个样本就是这个高维空间中的一个点。为了更好地在这个空间中采样，生成高质量的样本，本文提出了一种基于MCMC的采样策略。
整体的思想如下：定义一个大步长概率$p_{LS}$，以这个概率进行转移：
$$
T(\mathcal{u_i}\rightarrow\mathcal{v})=\left\{
  \begin{array}{ll}
    \mathcal{U}() & \text{以概率}p_{LS} \\
    Perturb(\mathcal{u_i}) & 其他情况
  \end{array}
\right.
$$
对于是否接受从样本$\mathcal{u_i}$跳到样本$\mathcal{v}$，本文使用loss与梯度的范数的乘积来决策，接受概率如下：
$$
\alpha(\mathcal{u_i}\rightarrow\mathcal{v})=\left\{
  \begin{array}{ll}
    1 & p(\mathcal{v})>p(\mathcal{u_i}) \\
    0 & 否则
  \end{array}
\right.
$$
也就是说引导生成一些不那么贴合目标函数的样本，以改进整个网络的性能。如下图：
![](19.png)
## 样本复用策略
与传统的神经网络生成一个固定大小的数据集并多次训练不同，本文边训练边生成新样本。
本文引入两个Loss，分别是用当前样本训练的$Loss_{exist}$和用新样本训练的$Loss_{new}$，当$Loss_{exist}$下降得比$Loss_{new}$快的时候，认为有过拟合的倾向，因此需要通过一个重用概率$p_{s}$来控制复用样本的频率来进行更多的新样本生成。本文采用了以下机制来决定$p_{s}$:
$$
p_s=\sigma(Loss_{exist}-Loss_{new}+\beta)
$$
其中$\sigma$是sigmoid函数


# 结果
# 总结
## 优点
+ 可变场景
+ 支持一些高频信息(焦散、阴影)
## 局限
+ 训练速度慢
+ 推理速度慢
+ 支持的可变参数有限