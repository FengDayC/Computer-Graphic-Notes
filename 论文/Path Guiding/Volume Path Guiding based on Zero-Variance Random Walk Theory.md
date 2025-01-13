# 方法
$$
L(x,\omega)=\underbrace{T(x,x_s)L(x_s,\omega)}_{L_s(x,\omega)}+\underbrace{\int_0^zT(x,x')\sigma_s(x')L_s(x',\omega)dt}_{L_m(x,\omega)}
$$
蒙特卡洛过程中需要做的决策一共有四种：
+ Scatter:是否计算体积内贡献，是只计算表面贡献还是计算表面+体积
+ Collision distance:下一个决策点的位置
+ Scattering Direction:下一个弹射方向
+ Termination:是否停止
![](论文/Path%20Guiding/pics/7.png)
零方差路径追踪是一种框架理论，意味着只要一个路径追踪器遵循这个理论，那么任何一个它产生的采样的方差就都是0。但是问题在于，要实现零方差，就需要知道光场信息(即每个采样点的入射辐射$L$以及接收到的内散射辐射$L_i$)，而这恰恰是需要路径追踪器求解的东西，这就会导致一个循环论证。为了避免这个循环论证逻辑，一般的零方差路径追踪会对$L$和$L_i$做一个假设来替代真实的值。
## 体积辐射度估计
一个球面函数可以用多个von Mises Fisher(vMF) lobe来拟合，表达式如下：
$$
V(\omega|\Theta)=\sum_{i=1}^K \pi_i\cdot v(\omega|\mu_i,\kappa_i)
$$
类似于高斯函数，每个lobe的两个参数一个决定了平均值，一个决定了分散程度。在路径引导中，使用这样的vMF可以避免在每个空间叶结点中存储方向信息，并且vMF之间的基本运算有解析解。
对于入射辐射的近似，采用如下形式进行：
$$
\begin{matrix}
\widetilde{L}(x,\omega)=\Phi(x)\cdot V_L(\omega|\Theta_L(x))\\
\Phi(x)=\int_XL(x,\omega')d\omega'
\end{matrix}
$$
对于内散射辐射，采用如下式子：
$$
\begin{matrix}
\widetilde{L}(x,\omega)=\Phi(x)\cdot \int_Sf(x,\omega,\omega')V_L(\omega'|\Theta_L(x))d\omega'\\
\end{matrix}
$$
如果将相位函数也用vMF表示，那么可以利用vMF的内积封闭性得到如下计算方式：
$$
\begin{matrix}
\widetilde{L}(x,\omega)=\Phi(x)\cdot V_{L_i}(\omega|\Theta_{L_i})\\
V_{L_i}(\omega|\Theta_{L_i})=(V_f*V_L)(\omega)
\end{matrix}
$$
也就是说，需要储存的vMF只有表示入射辐射的vMF
## 引导距离决策
采样距离的引导要做两个决策，一个是从下一个点在表面还是在体积内，如果在体积内的话，距离是多少。事实上，可以将这两个决策合并为同一个，即：下一个采样点的距离。采样pdf如下：
$$
p_d^{zv}(d|x,\omega)=\frac{T(x,x_d)\cdot\sigma_s(x_d)\cdot L_i(x_d,\omega)}{L(x,\omega)}
$$
要执行上述采样，一个简单的方法是将当前点到当前方向表面点的这段光路分为多个小段，对每一段计算上述pdf，累加得到一个cdf，并使用这一cdf决定在哪一段停止。但这样的方法有一个问题，即要得到整个cdf的归一化系数需要走完整个光路并计算每一段的pdf，这样的开销太大了。
为了解决这一问题，可以采用一个逐渐增加的cdf近似方法，如下：
$$
P_i(D\le d_{i+1})\approx\frac{1-T(d_i,d_{i+1})}{\sigma_t(d_i)}\cdot\frac{\sigma_s(d_i)\cdot\widetilde{L}_i(d_i)}{\widetilde{L}(d_i,\omega)}
$$
这实际上是出于一小段光路中发生散射的概率正比于内散射辐射和入射辐射之比。这一概率可以由以下式子产生：
$$
P(d_{i+1}<D)=\prod_{j=0}^{i}1-P_j(D\le d_{j+1})
$$
## 引导方向决策
方向决策的pdf如下：
$$
p_\omega^{zv}(\omega_{i+1}|\omega_i,x_{i+1})=\frac{f(\omega_i,\omega_{i+1})L(x_{i+1},\omega{i+1})}{L_i(x_{i+1},\omega_i)}
$$
这个概率密度函数来自[[Product Importance Sampling for Light Transport Path Guiding]],不同的是，这里使用vMF来近似。
事实上，上述两个采样都可以和普通的采样方法(距离采样基于透射距离(Transmittance)，方向采样基于相函数)进行线性插值以保证不准确的辐射估计不过度影响采样，如下：
$$
p=\alpha p^{zv}+(1-\alpha)p^{std}
$$
## 引导俄罗斯轮盘赌和分裂
俄罗斯轮盘赌策略通常采用albedo来控制停止，这通常会导致过快的停止而使方差增加(俄罗斯轮盘赌只能保证无偏，不能保证降低方差)。为了解决这一问题，可以将生存概率置为：
$$
q=\frac{E[R]}{I}
$$
其中$E(R)$为这条路径的潜在贡献，$I$为真实贡献。其中$I$可由weight window方法估计而来[[Adjoint-driven Russian roulette and splitting in light transport simulation]]
在体积渲染中，有两个地方需要采用RR,一个是距离采样后，一个是方向采样后。这两个采样的潜在贡献可以由对体积辐射度的估计计算而来。
体渲染中的分裂也存在与距离采样和方向采样两个地方，分裂事实上是由俄罗斯轮盘赌算法完成，不同之处在于，将$q$扩展至$[0,\inf)$，当$q<1$时执行轮盘赌，当$q>1$时执行分裂，根据$q$值来决定分裂的个数。
![](论文/Path%20Guiding/pics/8.png)