# PRT

## 核心思想
在实时渲染中，渲染方程一般写为三项：
$$
L(\vec{o})=\int_\Omega L(\vec{i})V(\vec{i})\rho(\vec{i},\vec{o})max(0,\vec{n}\cdot\vec{i}d\vec{i})
$$
将要积分的部分分为环境光照、可见性、BRDF三项，而这三项均可以描述为球面函数或者CubeMap

假设场景大体不发生改变，则对于任意一个着色点来说，上述可见性、BRDF项不发生改变（这两项称为光传输），可以进行预计算，变化的只有环境光照项


## PRT计算
将环境光照项进行球谐函数拆分：
$$
L(\vec{i})=\sum_i l_iB_i(\vec{i})
$$
### 漫反射项
对于漫反射的情况下，BRDF是一个常值：
$$
L(\vec{o})=\rho\int_\Omega L(\vec{i})V(\vec{i})max(0,\vec{n}\cdot\vec{i})d\vec{i}=\rho\sum_il_i\int_\Omega B_i(\vec{i})V(\vec{i})max(0,\vec{n}\cdot\vec{i})d\vec{i}
$$
由球谐函数的性质，第i个积分项相当于将可见性项投影到第i个基的方向上，可以进行预计算，从而：
$$
L(\vec{o})\approx\rho\sum l_iT_i
$$
亦即预计算如下的项：
$$
T_i\approx\int_\Omega B_i(\vec{i})V(\vec{i})max(0,\vec{n}\cdot\vec{i})d\vec{i}
$$
### 光滑项
与漫反射项不同，光滑项的BRDF并非一个常数（BRDF与视线方向有关）
因此，类似于漫反射项进行基分解时，分解出来的基与观察方向有关，即：
$$
L(\vec{o})\approx\sum l_iT_i(\vec{o})
$$
为了处理与$\vec{o}$相关的T项，继续做基分解：
$$
T_i(\vec{o})\approx\sum t_{ij}B_j(\vec{o})
$$
从而：
$$
L(\vec{o})=\sum\sum l_i t_{ij}B_j(\vec{o})
$$
可以预计算$t_{ij}$得到的矩阵，在渲染的时候计算光照拆分的向量与此矩阵进行点乘得到