# 摘要
在路径空间中进行平衡启发式MIS混合，以重用在训练中产生的样本

# 动机
之前的路径引导方法旨在通过$M-1$次训练得到一个pdf序列$p_1,p_2,...,p_M$，并用最后一次的pdf来引导最终的渲染结果。因此，这些方法主要的开销在于训练，而降低训练开销的主要方法是重用训练的采样结果。
像PPG这样的方法，在图像空间去重用了训练的采样结果，这就造成了一个问题，也就是图像空间

# 方法
本文的方法是在路径空间上去对不同的方法做MIS，换句话说，就是对$M$次迭代产生的pdf产生的所有采样，去进行MIS权重分配，公式如下：
$$
\left\langle \int_{\mathcal{P}} f(\overline{x}) \, d\overline{x} \right\rangle = \sum_{i=1}^{M} \frac{1}{n_i} \sum_{j=1}^{n_i} \frac{w_i(\overline{X}_{i,j}) \, f(\overline{X}_{i,j})}{p_i(\overline{X}_{i,j})}
$$
其中迭代了$M$次，每次产生的pdf为$p_i$，第i次pdf产生的第j条路径为$\overline{X}_{i,j}$。其中这条路径$\overline{X}=(x_1,...,x_N)$的pdf：
$$
p_i(\overline{x}) = p_i(x_1) \, p_i(x_2 \mid x_1) \, \prod_{j=2}^{N-1} p_i(x_{j+1} \mid x_j, x_{j-1})
$$
迭代$M$次后产生了若干条路径，全部记录下来，用以下方法计算MIS权重：
$$

$$
# 细节
