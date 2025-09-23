# 摘要

# 方法
## 光源选择pdf预测
对于直接光照来说，光照结果取决于多个光源的光照效果的叠加，即：
$$
L(x,\omega_o)=\sum_{y=1}^ML_y(x,\omega_o)=\sum_{y=1}^M\int_{A_y} F(x,z,\omega_o)dA_y(z)
$$
其中$z$是光源上的一点，F是影响因子，即：
$$
F(x,z,\omega_o)=L(z\rightarrow x)f(x,z\rightarrow x,\omega_o)G(x,z)
$$
事实上，对上式进行蒙特卡洛估计时，可以根据每个采样到的光源的影响程度分配一个pdf，则上式变为：
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
采用蒙特卡洛算法:\nabla_\theta D_{KL}(q(y)||p_\theta(y))=-\sum_{i=1}^N\frac{q(y)}{p_\theta(y)}\nabla_\theta logp_\theta(y)\\
q(y)\propto L_y(x,\omega_o)=\\

\end{array}
$$
## 光源簇选择
当场景的光源过多，只使用神经网络进行预测对网络能力要求太高