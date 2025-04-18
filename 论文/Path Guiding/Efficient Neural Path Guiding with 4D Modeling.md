# 摘要

# 方法
本文的目标是让网络学习一个带参数的PDF，即：
$$
p(\omega_i|\mathbf{x},\omega_o,\mathbf{u})\propto f_s(\mathbf{x},\omega_i,\omega_o,\mathbf{u})L_i(\mathbf{x},\omega_i,\mathbf{u})|cos\theta_i|
$$
用于进行路径引导。
## 空间特征场拆分
以一维为例，对于带参数的空间特征场拆分，本文使用了与[[Dynamic Neural Radiosity with Multi-grid Decomposition]]类似的拆分思路，即：空间位置作为一个单独特征场，空间位置的每个分量与参数的每一个分量也用单独特征场来表达对应关系。即：
$$
\begin{array}\\
g_\mathcal{X}(x)=G(x,y,z)\\
g_{\mathcal{X}\mathcal{U}}(x,u)=P_{XU_0}(x,u_0)\oplus P_{YU_0}(y,u_0)\oplus P_{ZU_0}(z,u_0)\\
\oplus ...\\
\oplus P_{XU_N}(x,u_N)\oplus P_{YU_N}(y,u_N)\oplus P_{ZU_N}(z,u_N) 
\end{array}
$$
也就是说，如果有$N$维向量，那么就需要使用$3N$个二维平面来表示整个空间特征场之间的相互关系。
## 渐进训练过程
如果场景参数变化很小，那么$p$的变化也应该不大。因此如果对于参数$u_1$有一个训练好的网络，那么可以通过微调得到表示一个与$u_1$相差不大的网络。因此，本文将整个$\mathcal{U}$切分成了很多小段，以一维为例，就是$[u_i,u_{i+1}]$,并且**假设在每个小段之间没有剧烈（高频）变化**。然后对于每一段，使用神经网络去预测这一段中的$p$的积分，即：
$$
p_{u_i}(\omega_i|\mathbf{x},\omega_o)\propto \int_{u_i}^{u_{i+1}}f_s(\mathbf{x},\omega_i,\omega_o,\mathbf{u})L_i(\mathbf{x},\omega_i,\mathbf{u})|cos\theta_i|du
$$
并且由于可以视作静态场景，因此无需采用神经特征场$g_{\mathcal{X},\mathcal{U}}(x,u)$去编码空间与场景参数的关系，只使用$g_\mathcal{X}(x)$来编码场景位置即可。
在训练的时候，在每个小段的开始会使用固定数目的采样点来对上一段的网络进行微调，也就是会更新神经特征场。随后，关闭神经特征场的更新，进行采样来训练这一个小段的网络。
## 优化
采用K-L散度的总和作为损失函数，KL散度的梯度的蒙特卡洛估计如下：
![](30.png)
# 结果
文章对比了PPG[[Practical Path Guiding for Efficient Light-Transport Simulation]]、NPM两个路径引导工作。
文章对比了两个场景（都只有一维场景参数）：动态模糊和谱渲染。
动态场景主要对比了NPM，NPM相比于本文方法而言，学到的分布在时间域上会产生一些“拖尾”。
# 结论
## 优势
+ 一定程度上可以处理带场景参数的“动态”场景
## 局限
+ 仍受限于场景参数的维度