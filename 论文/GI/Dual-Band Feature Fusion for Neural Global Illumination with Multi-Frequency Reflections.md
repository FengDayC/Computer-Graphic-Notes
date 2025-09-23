# 摘要
本文提出了一种带混合模块的神经网络，可以从First-Hit GBuffer以及Reflection GBuffer中学习到Glossy表面的光照信息，并且可以用在动态场景中

# 方法
![](32.png)
网络由三个部分组成，首先将First-Hit GBuffer以及Reflection GBuffer的信息编码为神经特征，即：
$$
(f_p,f_s)=(F(g_f),F(g_r))
$$
由于$f_p$和$f_s$得到的特征过于高频：一个是适用于纯漫反射表面的特征，一个是适用于纯反射表面的特征，因此引入了一个混合模块来解决Glossy表面的中间部分，即：
$$
f_{fused}=\mathcal{H}(f_p,f_s)
$$
最后使用一个解码器来将前面得到的特征进行解码，得到光照信息：
$$
L(x,\omega_o)=\mathcal{G}(g_f,f_{fused})
$$
## 逐物体的神经特征编码网络
对于场景中的每一个物体，采用类似[[Neural Global Illumination via Superposed Deformable Feature Fields]]的方式进行编码，即：
$$
f = \sum_i F_i(\mathcal{T}_i(g),r_i)=\mathcal{D}_i(\mathcal{T}_i(g_i),r_i,\mathcal{F}(x_i))
$$
其中，$\mathcal{D}$是一个由超网络推理出来的网络，$\mathcal{T}$表示将GBuffer变换到物体的局部坐标系中，$r$表示变换矩阵和物体的可变材质参数的组合$(M_i, v_i)$，$\mathcal{F}$表示三特征平面场
## 局部特征的混合网络
![](33.png)
左边部分是使用一个类似U-Net的卷积神经网络，输入GBuffer和上一步得到的特征去预测L个卷积核+权重以及一个权重向量$\gamma$，即：
$$
\{z^l,\beta^l\}_{l=0}^{L-1},\gamma=\mathcal{C}(g_f,g_r,f_p,f_s)
$$
预测出的L个卷积核用来对$f_s$进行卷积，并且对第k层卷积后的结果进行上采样，就得到第k+1层的性特征，再进行同样的操作得到k+2层的特征。由此可以得到L层特征，这些特征用预测出来的$\beta^l$进行混合得到总特征$\tilde{\mathcal{f}}_s$
预测出来的$\gamma$是插值权重，对得到的特征做如下插值得到最终的光照信息特征：
$$
f_{fused}=\tilde{\gamma_p}f_p+\tilde{\gamma_s}f_s+\tilde{\gamma_3}\mathcal{0}
$$
其中最后一项是0向量，是为了避免只在两个值之间进行插值所得到的结果在反射物体的边缘有不正常的轮廓。
# 训练
训练分为两个阶段，第一个阶段专注于训练网络对低频特征的捕获能力，第二个阶段训练对高频的捕获能力。
在低频阶段，关闭混合网络，并且把物体表面的粗糙度设置为不低于0.25，直接训练解码器和神经特征编码网络，并且使用一个较高的学习率使得网络收敛。
在高频阶段，去除所有限制，正常训练，使用的学习率较小。