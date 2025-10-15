# 摘要

# 方法
## 光源选择pmf预测
对于直接光照来说，光照结果取决于多个光源的光照效果的叠加，即：
$$
L(x,\omega_o)=\sum_{y=1}^ML_y(x,\omega_o)=\sum_{y=1}^M\int_{A_y} F(x,z,\omega_o)dA_y(z)
$$
其中$z$是光源上的一点，F是影响因子，即：
$$
F(x,z,\omega_o)=L(z\rightarrow x)f(x,z\rightarrow x,\omega_o)G(x,z)
$$
事实上，对上式进行蒙特卡洛估计时，可以根据每个采样到的光源的影响程度分配一个pmf，则上式变为：
$$
L(x,\omega_o)=\sum_{j=1}^N\frac{L_{Y_j}(x,\omega_o)}{p(Y_j)}
$$
类似于NPM[[Neural Parametric Mixtures for Path Guiding]]，采用KL散度为Loss对其进行优化。
其中目标分布为：
$$
q(y|x,\omega_o)=\frac{L_y(x,\omega_o)}{L_o(x,\omega_o)}
$$
$L_o$是所有光源在这一点上的影响结果，与NPM这样依赖KL散度和Adam优化器的方法相同，无需使用归一化的q即可进行优化。KL散度的梯度计算如下：
$$
\begin{array}\\
D_{KL}(q(y)||p_\theta(y))=\sum_{i=1}^Mq(y)\frac{q(y)}{p_\theta(y)}\\
\nabla_\theta D_{KL}(q(y)||p_\theta(y))=-\sum_{i=1}^Mq(y)\nabla_\theta logp_\theta(y)\\
采用蒙特卡洛算法:<\nabla_\theta D_{KL}(q(y)||p_\theta(y))>=-\frac{1}{N}\sum_{i=1}^N\frac{q(y)}{p_\theta(y)}\nabla_\theta logp_\theta(y)\\
q(y)\propto L_y(x,\omega_o)=\sum_j\frac{F(x,z_j,\omega_o)}{p(z_j|y)}\\
<\nabla_\theta D_{KL}(q(y)||p_\theta(y))>=-\frac{1}{N}\sum_{i=1}^N\frac{F(x,Z_i,\omega_o)}{p(Z_j|Y_j)p_\theta(Y_j)}\nabla_\theta logp_\theta(Y_j)\\
\end{array}
$$
## 光源簇选择
当场景的光源过多，只使用神经网络进行预测对网络能力要求太高。因此本文先采用先前文章中对光源进行聚簇的方法对光源进行组织(自底向上)，在第k层的结点打断，得到多个光源簇，网络预测选择这个光源簇的概率，再根据原有方法在光源簇中进行选择，从而两个概率相乘得到最终的光源权重
![](34.png)
在优化方面，光源选择pmf变为：
$$
p_{\theta}(Y_j)\rightarrow p_{\theta}(C_j)p(Y_j|C_j)
$$
则梯度变为：
$$
<\nabla_\theta D_{KL}(q(y)||p_\theta(y))>=-\frac{1}{N}\sum_{i=1}^N\frac{F(x,Z_i,\omega_o)}{p(Z_j|Y_j)p_\theta(C_j)p(Y_j|C_j)}\nabla_\theta logp_\theta(C_j)
$$
## 训练策略
如果采用随机初始化的神经网络，就需要非常多的样本使其收敛，不适合进行在线渲染。因此，本文先采用传统方法得到每个光源簇的概率估计，用神经网络学习这个估计与真实分布之间的差距，即先用传统方法得到一个权重$\omega_1,...\omega_S$，再以如下式子计算最终得到的权重：
$$
p_\theta(c)=\frac{e^{log\omega_c+f_\theta(x,\omega_o)[c]}}{\sum_{s=1}^S e^{log\omega_s+f_\theta(x,\omega_o)[c]}}
$$
