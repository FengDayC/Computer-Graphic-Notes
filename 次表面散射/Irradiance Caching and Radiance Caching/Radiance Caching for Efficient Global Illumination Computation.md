# 摘要
提出了一个算法框架，用球谐函数来转换BRDF和Radiance，实现了一个比Path Tracing更快、比Irradiance Caching更精确，更适合表示Glossy表面的光照的方法。

# 算法依据
+ Irradiance Caching擅于处理低频光照信息，而Path Tracing擅于处理高频光照信息，在低频和高频之间的光照信息，需要一个介于Irradiance Caching和Radiance Caching之间的方法，兼顾准确度和计算效率。
![](次表面散射/Irradiance%20Caching%20and%20Radiance%20Caching/pics/15.png)
+ 即便是处理Glossy的表面，临近的表面上接收到的Radiance也是缓慢变化的，与Irradiance的区别仅仅只是Radiance是具有方向性的（一个方向的光照，只会反射的方向不会过于分散）。

# 算法框架
整体的算法分为预处理和计算部分。预处理将表面的BRDF转换为球谐函数表示，计算部分将低频和高频的BRDF分别使用不同的处理方式，框架如下图：
![](次表面散射/Irradiance%20Caching%20and%20Radiance%20Caching/pics/16.png)

# Radiance Cache Record
一个Radiance Cache Record保存如下的信息：
+ 位置
+ uv+法线
+ 球谐函数系数用于描述接收到的Radiance
+ 平移梯度用于插值
+ 半球面上平均距离用于插值

# 球谐函数计算
将入射Radiance和BRDF都转换为球谐函数，保存为系数。利用系数相乘来计算出射Radiance。
## 转换BRDF
由于球谐函数适用于转换所有的（半）球面上的函数，因此可将BRDF用球谐函数进行转换：
![](次表面散射/Irradiance%20Caching%20and%20Radiance%20Caching/pics/17.png)
如第二个式子所示，执行一个积分即可预处理出来。
用户可以自定义一个球谐函数的最高次项和一个误差可接受范围，转换BRDF时，每个方向选取的球谐函数阶数$n$要不超过这个误差且最低。如果没有阶数满足这一要求，则放弃使用Radiance Caching算法。

## 转换入射Radiance
将入射Radiance转换为球谐函数系数表达，如下图：
![](次表面散射/Irradiance%20Caching%20and%20Radiance%20Caching/pics/18.png)
要求解系数，则利用正交性：
![](次表面散射/Irradiance%20Caching%20and%20Radiance%20Caching/pics/19.png)
![](次表面散射/Irradiance%20Caching%20and%20Radiance%20Caching/pics/20.png)

## 计算出射Radiance
有了经过插值求出的入射Radiance的系数表达以及特定方向BRDF的系数表达，两者进行简单点乘即可计算出特定方向的出射Radiance，如下图：
![](次表面散射/Irradiance%20Caching%20and%20Radiance%20Caching/pics/21.png)

# 梯度计算
使用球谐函数系数用于表示入射Radiance需要一种能够对这样的系数表示进行插值的方法。即计算向量$\lambda$的梯度$\nabla\lambda=[\frac{\partial\lambda}{\partial x},\frac{\partial\lambda}{\partial y},0]$，之所以只有两个方向，是因为这个梯度定义于表面的切向空间中
## 数值方法
用差分代替微分，将点进行平移之后分析贡献Radiance的平面的立体角，可以计算出如下式：
![](次表面散射/Irradiance%20Caching%20and%20Radiance%20Caching/pics/22.png)
示意图如下图：
![](次表面散射/Irradiance%20Caching%20and%20Radiance%20Caching/pics/23.png)

## 分析方法
对系数的定义式进行偏微分，并将立体角也看做一个函数：
![](次表面散射/Irradiance%20Caching%20and%20Radiance%20Caching/pics/24.png)
做偏微分：
![](次表面散射/Irradiance%20Caching%20and%20Radiance%20Caching/pics/25.png)
对基函数的偏微分如下：
![](次表面散射/Irradiance%20Caching%20and%20Radiance%20Caching/pics/26.png)
对立体角函数的偏微分如下：
![](次表面散射/Irradiance%20Caching%20and%20Radiance%20Caching/pics/27.png)
从而得到了求系数梯度的公式。

# 插值计算
对系数向量的插值方式如下：
![](次表面散射/Irradiance%20Caching%20and%20Radiance%20Caching/pics/28.png)
其中权重$w_i$的计算与[[A ray tracing solution for diffuse interreflection]]相同。
其中$S$集合即合法点的集合，要综合Irradiance Caching中权重的考虑以及对Radiance信息是否可用的考虑。
其中$\mathbf{R_i}$为旋转矩阵，将整个球谐函数系数旋转到要计算的点$\mathbf{p}$的方向，这一点也利用了球谐函数的性质。

# 总结
## 优点
计算效率高、精确度高，适合用于低频的BRDF
## 缺点
+ 旋转插值的时候会造成缺陷
![](次表面散射/Irradiance%20Caching%20and%20Radiance%20Caching/pics/29.png)
+ 球谐函数表示的BRDF，消耗的性能与BRDF的方向性有关