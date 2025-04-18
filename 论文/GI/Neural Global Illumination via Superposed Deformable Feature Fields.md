# 摘要
本文基于NeLT[[NeLT Object-Oriented Neural Light Transfer]]，提出了一种可以处理高频信息（焦散、阴影、高光）的全局光照方法。
# 方法
总体方法如下：
$$
L(\mathcal{x},\mathcal{\omega_o})=\mathcal{G}(\mathcal{A}({F_i(\mathcal{x_i},\mathcal{\omega_i};\mathcal{r_i})})
$$
$F_i$为描述所有物体的某种特征场，$r_i$为物体i面向对象的场景表示,$\mathcal{G}$是一个Decoder，$\mathcal{A}$是一种顺序无关的物体信息聚合方式
![pipeline](16.png)
## 面向对象的场景表示
定义对象-对象表示如下：
$$
\mathcal{r}_{ij}=\mathcal{R}_i(M_{ij},\mathcal{v_i})
$$
其中，$M_{ij}$表示物体i相对于物体j的变换矩阵，具体可以用其世界空间矩阵计算，如下：
$$
M_{ij}=M_i\times M_j^{-1}
$$
$\mathcal{R_i}$表示对象-全体对象的编码器，而$\mathcal{v_i}$表示变化的材质参数?
$r_{ij}$表示了物体i对于物体j的影响，而要得到整个场景对物体i的影响，可以简单地进行求和：
$$
\mathcal{r_i}=\sum_j\mathcal{r_{ji}}
$$
## 可形变神经特征场
为了得到一些更加高频的光照效果（阴影和焦散等），本文使用了一个可形变神经特征场来编码场景信息。这个神经特征场如下：
$$
F_i(\mathcal{x_i},\mathcal{\omega_i};\mathcal{r_i})=\mathcal{D_i}(\mathcal{F_i}(\mathcal{x_i}+\mathcal{T_i}(\mathcal{x_i};\mathcal{r_i})_p),\mathcal{T_i}(\mathcal{x_i};\mathcal{r_i})_h,\mathcal{g_i};\mathcal{r_i})
$$
$\mathcal{D_i}$是一个解码器，其输入除了GBuffer$g_i$和物体表示$r_i$外，还有一个特征场$\mathcal{F_i}$和辅助特征$\mathcal{T_i}$。其中$T_i$输入场景表示和采样点，输出的前三个通道为采样点的位置偏移，记为$\mathcal{T_i}(x_i;r_i)_p$剩余的通道记为$\mathcal{T_i}(x_i;r_i)_h$作为辅助特征。
总的神经特征场生成流程如下图：
![](17.png)
首先将上一步得到的场景表示喂给两个超网络，分别生成$\mathcal{T_i}$和$\mathcal{D_i}$的参数；将GBuffer变换到物体局部坐标系中，位置信息单独拿出来输入$\mathcal{T_i}$得到位置偏移和辅助特征；位置偏移和位置加和之后输入三平面特征场，采样得到特征，将其于辅助信息、剩余的GBuffer一起拼接起来输入解码器$\mathcal{D_i}$得到单个物体的影响向量。最后将所有物体的影响向量拼接在一起输入一个大的解码器得到GI结果。
### 三平面特征场
上式中的$\mathcal{F_i}$即为三平面特征场，定义如下：
![](15.png)
这是一个三维向量场，其中输入为一个三维点$\mathcal{x}=(\mathcal{x_0},\mathcal{x_1},\mathcal{x_2})$，输出是一个向量，这个向量的长度为$3m$也就是由$m$个三维向量构成的，$m$是分辨率等级。
而$\mathcal{P}^*_{i,l}$表示的是一个可被优化的**二维曲面**，$l$表示分辨率等级（类似于mipmap）。
$\mathcal{\omega_j}(\alpha)$表示的是被参数$\alpha$所控制的窗口函数，这个窗口函数的设置是为了控制权重的更新层级：即在刚开始训练的时候$\alpha$很小，导致更新只在低分辨率等级上进行，有利于学习大的形变，而随着训练逐渐增加$\alpha$的值，致使更高等级分辨率的参数得到不断更新。
# 实现
网络结构略
# 总结
## 创新点
+ 利用一个可形变的神经特征场得到对高频信息的更好表达
+ 使用对象-对象的数据表示以并行编码并且渲染顺序无关
## 局限
+ 动态物体：无法很好地处理大量动态物体的问题
+ 大尺度场景：由于采用了三平面的神经特征场来描述场景信息，对于大尺度的场景开销会比较大，难以兼顾细节
+ 泛化性：对于训练集中未出现的高频信息，该方法难以捕捉