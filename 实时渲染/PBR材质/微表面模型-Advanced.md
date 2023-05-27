# 微表面模型-Advanced
[[基于物理的材质]]

## Distribution of Normals
法线分布函数有着很多不同的模型描述

### Beckmann NDF
数学表达如下：
$$
\begin{matrix}
D(\vec{h})=\frac{e^{-\frac{tan^2\theta_h}{\alpha^2}}}{\pi\alpha^2cos^4\theta_h}\\
\alpha:描述表面粗糙程度的参数\\
\theta_h:半程向量的角度
\end{matrix}
$$
理解：
+ 上述表达式类似于高斯分布函数的形式，这里的$\alpha$类似于高斯分布的$\delta$项（也就是方差）
+ 这里的$\theta$表示的是半程向量与法线之间的角，表示这个公式是各性同向的
+ 一般认为Beckman NDF定义在坡度空间上，坡度空间(slope space)是指在单位半球的顶端的切平面上的空间，也就是说Beckman是坡度空间上的一个高斯函数。这意味着此分布的定义域为一个无限集，此平面上的每一个点都对应一个角度
+ 分母的项是为了保证在半球投影在圆面上的空间中积分为1（能量守恒性质）

### GGX(Trowbridge-Reitz) NDF
GGX也是一种类似于高斯函数的NDF，但与Beckman不同，GGX具有“长尾性”，也就是说其在自变量偏移均值较大的时候其衰减得会比较慢，因此其在边缘过渡处的处理会有光晕般的柔和

### Generalized Trowbridge-Reitz NDF
一种加入了一个可调整参数的GGX模型，可以对GGX进行调整其尾巴的长度

## Shadow-Masking(Geometry) Term
几何项描述了微表面的自遮挡现象，防止了在物体边缘出现的白圈（法线与视线近乎直角的时候，微表面公式的分母的$\vec{n}\cdot\vec{i}\approx0$）

### 一些定义
Shaodwing：微表面挡住了入射的光源
Masking：微表面挡住了出射光线（挡住了视线）

### Smith Shadow-Masking Term
此模型将shadowing和masking分开考虑

## 白炉测试
在一个全白各向相同的环境光下，应该完全看不到其中的一个白色的物体（因为微表面模型在全白的情况下应该没有能量损失）
这样的测试称为白炉测试，是用来衡量一个BRDF的能量损失的标准

朴素的微表面模型粗糙度越高的物体在白炉测试中表现越差，是因为通常的模型只考虑了一次光线反射，由于自遮挡造成了能量损失，而现实情况中光线在被自遮挡后还会无限弹射下去

## Kulla-Conty Approximation
考虑白炉的情况，渲染方程的$L$项等于1，从而对于任一出射方向：
$$
E(\mu_o)=\int_0^{2\pi}\int_0^{\frac{\pi}{2}}f(\mu_o,\theta,\phi)cos\theta d\theta d\phi
$$
则由于微表面模型只计算一次弹射而损失的能量为：
$$
1-E(\mu_o)
$$
于是需要将这个能量补上就能够得到正确的结果
于是就需要一种BRDF，其积分出来的结果刚好为$1-E(\mu_o)$
且由于BRDF的对称性,此BRDF拥有因式$(1-E(\mu_i))$
记为：$BRDF=c(1-E(\mu_i))(1-E(\mu_o))$
由上式可以推出$c$的取值
$$
\begin{matrix}
c=f(\mu_o,\mu_i)=\frac{(1-E(\mu_o))(1-E(\mu_i))}{\pi(1-E_{avg})}\\
E_{avg}=2\int_0^1E(\mu)\mu d\mu
\end{matrix}
$$

这样得出的BRDF与原BRDF相加，在进行积分，就能够得到正确的结果(补全能量)

为了尽快的算出这个补偿的BRDF，可以采用预计算的方法，这个补偿函数的值只与$\mu_o$与$soughness$有关，因此就可以进行预计算的生成

而对于非白炉的情况（物体具有颜色的情况）K-C方法先考虑没有颜色的情况，计算出正确的能量，再计算颜色的能量损失

定义一个平均菲涅尔项：
$$
F_{avg}=\frac{\int_0^1F(\mu)\mu d\mu}{\int_0^1\mu d\mu}=2\int_0^1F(\mu)\mu d\mu
$$
这个平均菲涅尔项表明了光线经过一次反射，会有多少的能量被反射出来（对于有颜色的物体，这个值小于1）

由此有以下推导：
$$
\begin{matrix}
直接能够看到的光线：F_{avg}E_{avg}\\
反射k次看到的光线：F_{avg}^k(1=E_{avg})^k\cdot F_{avg}E_{avg}\\
进行级数求和：\frac{F_{avg}E_{avg}}{1-F_{avg}(1-E_{avg})}
\end{matrix}
$$
