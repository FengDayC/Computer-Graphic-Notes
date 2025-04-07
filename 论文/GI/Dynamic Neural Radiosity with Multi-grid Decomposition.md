# 摘要
本文扩展了[[Neural Radiosity]]，将其改进到更好的支持动态场景。
# 方法
本文要解决的问题：给定点、视线方向和场景动态参数，预测对应的辐射度
$$
L(x,\omega_o)\rightarrow L(x,\omega_o,\mathcal{v})
$$
## 空间特征场拆分
和Neural Radiosity类似，本文也使用多个不同分辨率的空间特征场去局部化一些网络参数。本文的空间特征场是这样的：一共有$L$个不同分辨率的特征场，其中每个特征场的格点上保存$F$维特征。计算空间中某一点的特征时，采样每个分辨率的特征场并拼接到一起。
$$
G(x)=\overset{L}{\underset{i=0}{\Huge{\oplus}}}trilinear(x,V_l[x])
$$
如果考虑动态场景因素，有$n$个动态变量，可以使用$n+3$维的特征场。但这样的特征场太耗费空间了，本文将这个特征场拆分为若干低维特征场。
拆分为三部分：
+ 1个空间位置相关特征：$f_{\mathcal{X}}(x)$
+ $3n$个二维特征面：$n$个场景变量，每个场景变量对应空间位置的三个分量xyz，得到$3n$个二维平面$f_{\mathcal{X}\mathcal{V}}(x,v)$
+ 一个MLP表示的各个场景变量之间的相关关系$f_{\mathcal{V}}(v)$
其中$f_{\mathcal{V}}(v)$定义如下：
$$
f_{\mathcal{V}}(v)=MLP(Freq(v))=MLP(...,cos(2^0v_i),sin(2^0v_i)...,cos(2^Lv_i),sin(2^Lv_i),...)
$$
和Nerf类似，在输入MLP之前，将特征向量进行频率编码
## Pipeline
![](24.png)
输入场景变量$v$，位置$x$，以及用方向计算的球谐基的值，加上一些额外输入，网络推理出辐射度。
## 优化方法
和[[Neural Radiosity]]类似，本文的优化目标也是LHS与RHS之差，即：
$$
r_\theta(x,\omega_o,v)=\underbrace{L_\theta(x,\omega_o,v)}_{\text{Left Hand Side}}-\underbrace{(L_e(x,\omega_o,v)+\int_{\mathcal{H}^2}f(x,\omega_i,\omega_o)L_\theta(x',-\omega_i,v)(n\cdot\omega_i)d\omega_i)}_{\text{Right Hand Side}}
$$
在优化过程中，残差的计算也是使用相同的方法。对于LHS，直接采样一个随机位置和随机方向。对于RHS，通过蒙特卡洛方法进行估计。
## 一些细节
+ 为了使得网络更好地学习，实际使用的时候额外输入了视线方向的镜面出射方向(更确切地说是这一方向的球谐函数值)
+ 几何信息(albedo、normal等)使用OneBlob encoding后再输入网络
+ 随着训练代数的增加，逐步增加对RHS进行蒙特卡洛估计的采样数
# 实现
## Specular处理
与Neural Radiosity一致，对Specular进行追踪直至Diffuse再进行预测。对于可能存在多个Lobe的BSDF，本文没有处理
# 结果
本文对比了PT、Oidn、AE、NeLT、Coomans以及原方法NR
+ 对比AE，渲染更快且结果更好。
+ 对比NeLT，性能更好。
+ 对比Coomans，可变场景参数更多
+ 对比微调NR，更不用说了
# 结论
## 优点
+ 推理速度快（由于拆分了场景参数）
+ 质量不错
+ 支持动态参数场景
## 局限
+ LHS采样存在噪声，尤其是Specular