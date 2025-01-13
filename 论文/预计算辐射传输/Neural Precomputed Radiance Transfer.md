# 摘要
基于静态场景+动态全局光照的假设，提出和对比了四种神经网络结构来获取预计算全局光照。输入环境观贴图和G-Buffer，通过网络推理得到实时环境光照结果。

# 数学依据
PRT将光线传播过程使用算子建模为三个部分：
$$
L=RSE
$$
即：最后一次反射、传播矩阵和光源
其中，反射可根据BRDF分为漫反射和光滑反射，因此有：
$$
L=R_DSE+R_SSE
$$

# 算法
![](论文/预计算辐射传输/pics/9.png)
如上图，论文研究了四个网络结构
## 环境光编码
类似于传统PRT将环境光通过基函数编码为一个向量，使用一个CNN来对环境光进行编码：
$$
\hat{e}=\mathcal{F}_L(E)
$$
## Baseline
第一种结构直接使用神经网络求解Radiance，输入着色点信息(除了渲染方程参数之外，还输入了反射方向以方便神经网络进行理解)以及环境光编码：
$$
L=\hat{\mathcal{S}}(\hat{\mathbf{e}},\mathbf{x},\mathbf{\omega}_o,\mathbf{\omega}_r,\mathbf{n},k_d,k_s,\alpha)
$$
网络结构只使用多层感知机和ReLU函数
## PRT-Inspired
第二种结构参考PRT的流程，将传播过程（传播矩阵）单独采用一个神经网络表示，即：
$$
\hat{\mathbf{s}}=\mathcal{F}_T(\mathbf{x},\mathbf{\omega}_o,\mathbf{\omega}_r,\mathbf{n},k_d,k_s,\alpha)
$$
传播过程采用多层感知机训练。综合这个传播结果与环境光编码，得到最后的光照结果：
$$
L=\Phi(\hat{\mathbf{s}},\hat{\mathbf{e}})
$$

## Albedo Factorized
第三种结构和第二种一样，将传播过程单独分离出来，但求解光照的算子输出两个diffuse和specular两个光照结果，最后将他们线性组合得到最终的结果
$$
\begin{matrix}
(L_D,L_S)=\Phi(\hat{\mathbf{s}},\hat{\mathbf{e}})\\
L=k_dL_D+k_sL_S
\end{matrix}
$$
## Diffuse-Specular Separation
最后一种结构将Diffuse和Specular的计算彻底分开，分别用不同的网络进行拟合，公式如下：
$$
\begin{matrix}
L=k_d\Phi_D(\hat{\mathbf{s}}_D,\hat{\mathbf{e}})+k_s\Phi_S(\hat{\mathbf{s}}_S,\hat{\mathbf{e}})\\
\hat{\mathbf{s}}_D=\mathcal{F}_T^D(\mathbf{x},\mathbf{n},k_d)\\
\hat{\mathbf{s}}_S=\mathcal{F}_T^S(\mathbf{x},\mathbf{n},k_s,\mathbf{\omega}_o,\mathbf{\omega}_r,\alpha)\\
\end{matrix}
$$

## 训练数据
每个场景使用3000张使用Falcor引擎渲染的256x256的图片(包含图像和GBuffer)用于训练以及200张400x400的图片用于验证
每个场景随机放置摄像机位置和随机选择摄像机方向
# 结果
## 不同方法的对比
1. baseline方法难以捕获光源/视角相关的特性（因为他的输入是局部的，因此很难从局部信息推断出光源无关的光线传输结果），例如高光、阴影和多次反射光，因此baseline方法在有着更多的与时间相关的artifact。PRT-Inspired方法通过将光线传播分开来，很好地解决了这一问题。
2. 后面两个方法，分开了漫反射项和光滑项，使得网络可以学习不受albedo影响的辐射分布，并且更加连贯。
3. 最后一个完全分开的方法:Diffuse网络更专注于位置相关的输入，而Glossy网络更专注于方向相关的输入
从算法准确度来说，完全分开的方法与Ground Truth的差别最小：
![](论文/预计算辐射传输/pics/10.png)
## 与现有方法的对比
1. 与实时光追对比：远不如Ours
2. 与Neural Radiance Caching对比：不如Ours
3. 与Deep Shading's U-Net对比：baseline效果好于U-Net
4. 与原始PRT对比：远好于

# 局限
1. 效果依赖于训练输入的摄像机位置和视角，可能可以通过更好地布置摄像机的策略解决

# 思考
+ 第四种方法要将光线传播过程$\hat{s}$进行了拆分，这是不太有道理的，因为diffuse和specular仅仅在最后一步反射有所区别，而各个方向入射的光线是没有区别的，因此有可能只对最后一步反射函数$\Phi$进行区分效果会更好
+ 光线传播过程$S$中蕴含了整个场景的几何信息，单使用G-Buffer的信息来训练可能会让网络无法正确预测场景中的几何关系（例如多重反射和遮挡）