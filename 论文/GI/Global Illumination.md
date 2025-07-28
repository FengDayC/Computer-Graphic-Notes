# 核心问题
对于一个场景，求解辐射度：
$$
L(x,\omega)
$$
# 前置文献
## [[Instant Neural Graphics Primitives with a Multiresolution Hash Encoding]]

# 传统渲染
## 屏幕空间

## 世界空间
### [[Dynamic Voxel-Based Global Illumination]]

# 神经渲染
## 编码对象
### [[NeLT Object-Oriented Neural Light Transfer]]
思想：通过一个个添加对象，编码GI的变化函数，得到GI结果
限制：
+ 推理时间较长
+ 无法解决Deformable的物体
+ 对高频信息的还原有限
### [[Neural Global Illumination via Superposed Deformable Feature Fields]]
思想：基于NeLT，通过一个可形变的神经辐射场来拟合高频的场景信息从而实现对焦散、阴影等效果的更好实现
限制：
+ 大尺度场景下收到分辨率的限制
+ 多动态物体影响推理效率
## 编码场景信息
### [[Active Exploration for Neural Global Illumination of Variable Scenes]]
思想：采用一个统一的大网络对GI信息进行编码，输入GBuffer，输出GI结果。在训练阶段，通过对场景变量的主动探索，边训练边生成样本，引导网络学习更多有效样本。(与PG的思路有点像)
限制：
+ 采用一个统一大网络编码所有场景信息，导致训练速度、推理速度受到影响，并且影响网络对特征的学习。
### [[Neural Radiosity]]
思想：通过将渲染方程两边未知量替换为神经网络预测结果，计算梯度并优化整个系统得到正确的光照结果。
限制：
+ 推理时间较久局限于静态场景
+ 神经网络对于高频信息学习有限
### [[Dynamic Neural Radiosity with Multi-grid Decomposition]]
思想：通过将NR中的空间特征场进行拆分，实现了对低维动态场景的支持。
限制：
+ 存在一定噪声，尤其是高频信息
+ 局限于低维动态场景
### [[Real-time Neural Rendering of Dynamic Light Fields]]
思想：加入了时间变量，实现了对1维动态场景的支持
限制：
+ 动态场景变化小

## 纯神经渲染
### [[RenderFormer-Transformer-based Neural Rendering of Triangle Meshes with Global Illumination]]


## 神经探针
### [[Neural Light Grid]]
+ 直接用神经网络编码光照探针

# 未解决的问题
+ 多动态物体场景
+ 训练时间较长

# 一些想法
## Neural Rendering的本质
针对合成场景神经渲染本质上是一种缓存和压缩的方法。相当于已知整个辐射场，将其压缩到一个轻量的神经网络中，在渲染时通过神经网络恢复压缩前的数据，查询这个辐射场的原始信息。但与传统的缓存方法不一样的是，神经网络可以自行脑补一部分从未求解过的辐射场信息（类似于传统方法中的插值）。
## Neural Radiosity系列方法
+ 用某种方法单独编码可见性（或者说形状因子）
+ Neural Radiosity的场景辐射信息是一步步从光源扩散到整个场景中的，因此在训练上可能可以利用这一点，而非随机进行采样
+ 更具方向性的优化过程
	+ 引导优化：类似于PPG，储存一部分的可信信息到场景中，引导后面的训练
	+ 双向优化：从光源出发对场景进行优化
+ Specular表面的处理
+ NR与其他方法的区别：NR更多的是一种强化学习的思路，而非传统的监督学习思路
## 动态场景的处理方式
+ 有限动态：把所有能动的部分全部使用插值的形式表达，插值参数组成一个场景向量
+ 预定动态：直接规定每一个时刻，场景每部分的位置

$$
r_\theta(x,\omega_o,v)=\underbrace{L_\theta(x,\omega_o,v)}_{\text{Left Hand Side}}-\underbrace{(L_e(x,\omega_o,v)+\int_{\mathcal{H}^2}f(x,\omega_i,\omega_o)L_\theta(x',-\omega_i,v)(n\cdot\omega_i)d\omega_i)}_{\text{Right Hand Side}}
$$

$$
\begin{array}\\
RHS = \sum_{j=1}^N\frac{f(x,\omega_{ij},\omega_o)L_\theta(x',-\omega_{ij})(n\cdot\omega_{ij})}{pdf(x,\omega_{ij})}\\
pdf(x',\omega_{ij})\propto L_\theta(x',-\omega_{ij})
\end{array}
$$

