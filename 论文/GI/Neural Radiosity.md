# 摘要
提出了一种“自训练”的GI方法，直接预测辐射度，适用于静态场景。
# 方法
## 问题转化
本文方法是直接预测给定着色点的辐射度，即使用一个神经网络直接算出$L_\theta(x,\omega_o)$项。（其中$\theta$是网络参数）
将渲染方程两边的辐射度(要求解的未知量)都用这一网络表示，定义残差：
$$
r_\theta(x,\omega_o)=\underbrace{L_\theta(x,\omega_o)}_{\text{Left Hand Side}}-\underbrace{(L_e(x,\omega_o)+\int_{\mathcal{H}^2}f(x,\omega_i,\omega_o)L_\theta(x',-\omega_i)(\omega_i\cdot \mathcal{n})d\omega_i)}_{\text{Right Hand Side}}
$$
目标即使得Left Hand Side=Right Hand Side.
将所有表面的残差的L2范数之和定义为Loss：
$$
\mathcal{L}(\theta)=||r_\theta(x,\omega_o)||_2^2=\int_{\mathcal{M}}\int_{\mathcal{H}^2}r_\theta^2(x,\omega_o)d\omega_odx
$$
整个问题变为一个优化问题：
$$
\hat{\theta}=argmin_\theta\mathcal{L(\theta)}
$$
## 梯度计算
首先，可以对$\mathcal{L}(\theta)$使用如下蒙特卡洛估计：
$$
\mathcal{L}(\theta)\approx\frac{1}{N}\sum_{j=1}^N\frac{r_\theta^2(x_j,\omega_{o,j})}{p(x_j,\omega_{o,j})}
$$
即随机采样一些表面点，以及随机采样一些表面点半球上的方向，计算残差并除以概率密度，得到对总残差的估计。
对其求梯度有：
$$
\nabla_\theta\mathcal{L}(\theta)\approx\frac{1}{N}\sum_{j=1}^N\frac{2r_\theta(x_j,\omega_{o,j})\nabla_\theta r_\theta(x_j,\omega_{o,j})}{p(x_j,\omega_{o,j})}
$$
其中残差的梯度是：
$$
\nabla_\theta r_\theta(x_j,\omega_{o})=\nabla_\theta L_\theta(x,\omega_o)-\int_{\mathcal{H}^2}f(x,\omega_i,\omega_o)\nabla_\theta L_\theta(x',-\omega_i)(\omega_i\cdot\omega_o)d\omega_i
$$
用蒙特卡洛积分：
$$
\nabla_\theta r_\theta(x_j,\omega_{o,j})=\nabla_\theta L_\theta(x_j,\omega_{o,j})-\frac{1}{M}\sum_{k=1}^M\frac{f(x_j,\omega_{i,j,k},\omega_{o,j})\nabla_\theta L_\theta(x',-\omega_{i,j,k})(\omega_{i,j,k}\cdot\omega_{o,j}))}{p(\omega_{i,j,k})}
$$
## 训练过程
有了梯度，就可以采用随机梯度下降法对网络参数$\theta$进行求解，求解算法如下：
![](21.png)
即：采样一系列的表面点以及方向，计算梯度，用梯度更新参数。
上述过程，对于每个采样点，直接输入网络计算LHS，，然后在这个采样点的半球面上采样，进行一次光线求交，得到的交点输入网络计算RHS，**无需一路追踪到光源**
# 实现细节
## 规范化残差
为了避免数据范围过大导致网络的数值不稳定，文章中的Loss实际上使用了规范化后的残差：
![](22.png)
## 对Emission重新参数化
文章发现网络如果考虑预测自发光项比较困难，因此将自发光项从网络预测中剔除，重新将残差表示如下：
$$
r_\theta(x,\omega_o)=\underbrace{L_\theta(x,\omega_o)-L_e(x)}_{N_\theta(x,\omega_o)}-(\int_{\mathcal{H}^2}f(x,\omega_i,\omega_o)(\underbrace{L_\theta(x',-\omega_i)-L_e(x)}_{N_\theta(x',\omega_o)}+L_e(x))(\omega_i\cdot\omega_o)d\omega_i)
$$
即：
$$
r_\theta(x,\omega_o)=N_\theta(x,\omega_o)-(\int_{\mathcal{H}^2}f(x,\omega_i,\omega_o)(N_\theta(x',\omega_o)+L_e(x))(\omega_i\cdot\omega_o)d\omega_i)
$$
## 多分辨率特征网格
文章尝试了直接采用一个MLP来表示网络，但是发现其难以学习一些局部特征，如高频阴影等。因此，文章采用了特征网格，将一些特征与位置信息关联起来。
对于一个查询点，先查询其附近的特征网格，得到的结果作为网络输入的一部分。并且本文中使用了多个分辨率的特征网格，以拟合局部的和全局的信息。
## 网络输入以及额外特征
为了使更好地学习，网络接受的所有输入如下：
![](23.png)
## Specular表面处理
模型不是很能学习到Specular表面这样的高频信息，如果光线Hit到了Specular表面，他就会进行光线追踪直至到一个Non-Specular表面再进行网络查询。
## 网络结构
整个网络由6个宽度为512的全连接层构成，用ReLU连接。
# 结果
## LHS/RHS
实验得到，LHS和RHS的差距不大，并且远快于Path Tracing
## 消融实验
特征网格对于学习阴影信息非常重要。
## 动态场景
对于动态场景，需要对网络进行微调才能得到正确的结果，基本上可以认为本文方法不支持动态场景。
# 结论
## 优势
+ 无需光线追踪，无需GT，自训练的网络
+ 对Specular有着不错的表现，但是会增加性能开销
## 劣势
+ 渲染速度慢
+ 不支持动态场景