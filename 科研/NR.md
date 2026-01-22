$$
\begin{array}\\
L(x,\omega_o)=\underbrace{L_e(x,\omega_o)+\int_{\mathcal{H}^2}f(x,\omega_i,\omega_o)L_\theta(x',-\omega_i)(\omega_i\cdot \mathcal{n})d\omega_i}_{\text{Right Hand Side}}\\
\approx L_e(x,\omega_o)+\frac{\int_{\mathcal{H}^2}L_\theta(x',-\omega_i)d\omega_i}{\int_{\mathcal{H}^2}d\omega_i}\int_{\mathcal{H}^2}f(x,\omega_i,\omega_o)(\omega_i\cdot \mathcal{n})d\omega_i\\
\end{array}
$$
$$
\begin{array}\\
L_\theta(x,\omega_o)=L_e(x,\omega_o)+\int_{\mathcal{H}^2}f(x,\omega_i,\omega_o)L_\theta(x',-\omega_i)(\omega_i\cdot \mathcal{n})d\omega_i\\
L(x,\omega_o)=L_\theta(x,\omega_o)+r(x,\omega_o)\\
\hat r(x,\omega_o)=L_\theta(x,\omega_o)-L_e(x,\omega_o)-\int_{\mathcal{H}^2}f(x,\omega_i,\omega_o)L_\theta(x',-\omega_i)(\omega_i\cdot \mathcal{n})d\omega_i\\
\hat r_2(x,\omega_o)=L_\theta(x,\omega_o)-L_e(x,\omega_o)-\int_{\mathcal{H}^2}f(x,\omega_i,\omega_o)L(x',-\omega_i)(\omega_i\cdot \mathcal{n})d\omega_i\\
=L_\theta(x,\omega_o)-L_e(x,\omega_o)-\int_{\mathcal{H}^2}f(x,\omega_i,\omega_o)L_\theta(x',-\omega_i)(\omega_i\cdot \mathcal{n})d\omega_i-\int_{\mathcal{H}^2}f(x,\omega_i,\omega_o)r(x',-\omega_i)(\omega_i\cdot \mathcal{n})d\omega_i\\
=\hat r(x,\omega_o)-\int_{\mathcal{H}^2}f(x,\omega_i,\omega_o)r(x',-\omega_i)(\omega_i\cdot \mathcal{n})d\omega_i\\
\end{array}
$$
$$
\begin{array}\\
p_0(\omega_i)\propto L_\theta(x,\omega_i)\\
p_1(\omega_i)=1-p_0(\omega_i)\\
P(x,i)=1-\frac{L_\theta(x,\omega_i)}{\sum_jL_\theta(x,\omega_j)}
\end{array}
$$
$$
p=\sigma(loss-loss_{avg}+\beta)
$$
# 主要问题
无法找到一个数学上说得通的优化思路
# 解决局部最小的问题
## AE
考虑loss和梯度的大小的乘积
## 场景
使用静态场景测试当前的算法

# 基于残差和梯度的样本复用
$$
weights(x) = r(x)\cdot||\nabla_{\theta} L_\theta(x)||\propto\nabla_{\theta}
$$
$$
p(u\rightarrow v) = \frac{w(v)}{w(u)}
$$
\alpha(u\rightarrow v)=\begin{cases}
1 & w(v)>w(u)\\
0 & otherwise
\end{cases}
$$
sample=
$$

# 理论推导
对于想要拟合的函数$f(x)$，其满足以下性质：
$$
f(x) = e(x) + \int f(x)s(x)dx
$$
以网络参数$\theta$来定义：
$$
f_\theta(x) = e(x) + \int f_\theta(x)s(x)dx
$$
残差计算为：
$$
r_\theta(x)=f_\theta(x) -( e(x) + \int f_\theta(x)s(x)dx)
$$
以此残差更新网络，得到最终的结果

$$
\begin{array}\\
p(u\rightarrow v)=sigmoid(L_u-L_v+\beta)\\
\end{array}
$$
$$
Loss_{reuse}=\sum (Loss_{now}-Loss_{avg})logp
$$
