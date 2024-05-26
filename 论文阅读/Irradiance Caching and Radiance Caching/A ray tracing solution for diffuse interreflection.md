# 摘要
提出了一种将间接光照结果存储在物体表面来加速光线追踪计算的方法。

# 算法主要依据
+ 计算Irradiance需要发射很多的光线
+ Irradiance的值是与视角无关的
+ Irradiance的值变化相对缓慢

# 算法框架
针对一个着色点，分为两种情况：
1. 这一着色点周围存在已经完成计算并保存的着色点->直接使用周围着色点保存的信息或进行插值(Secondary Method)
2. 这一着色点周围没有保存的信息->调用蒙特卡洛算法计算这一着色点的值(Primary Method)
![](论文阅读/Irradiance%20Caching%20and%20Radiance%20Caching/pics/1.png)
# Primary Method
蒙特卡洛积分，没什么好说的

# Secondary Method
## Illuminance Gradient
为了更好地确定Irradiance的变化程度，需要对Irradiance场进行梯度的计算，将Irradiance场看作函数：$E(\vec{x},\vec{\xi})$，那么有：
![](论文阅读/Irradiance%20Caching%20and%20Radiance%20Caching/pics/2.png)
即对$E$二维泰勒展开的一次项，计算得到：
![](论文阅读/Irradiance%20Caching%20and%20Radiance%20Caching/pics/3.png)
使用向量的方法，可以替换为：
![](论文阅读/Irradiance%20Caching%20and%20Radiance%20Caching/pics/4.png)
将这个误差的倒数作为权重用于插值：
![](论文阅读/Irradiance%20Caching%20and%20Radiance%20Caching/pics/5.png)

## 插值实现细节
+ 常量$a$用于控制能够接受的最大估计误差
+ 为了更加精确的筛选用于插值的着色点，引入参数
![](论文阅读/Irradiance%20Caching%20and%20Radiance%20Caching/pics/6.png)
当$d_i(\vec{P})<0$时，拒绝点参与插值。
以防止下图的情况出现，即不让“背后”的点参与插值计算：
![](论文阅读/Irradiance%20Caching%20and%20Radiance%20Caching/pics/7.png)
## 光照信息的存储和搜索
为了更快地搜索保存的光照信息，使用一棵八叉树进行存储。
存储时，遵循如下原则：每个光照信息记录点保存在符合如下条件的结点中：
+ 包含这个光照信息点
+ 结点管辖区域边长在$2aR_i$到$4aR_i$之间（这样既保证了节点区域内的所有点在光照信息点的"合法区域"内，又保证了这一层级的兄弟节点管辖的区域不会全部位于这个光照信息点的“合法区域”内）

保存在八叉树内之后，执行邻近的搜索过程如下：
![](论文阅读/Irradiance%20Caching%20and%20Radiance%20Caching/pics/8.png)
# 总结
## 优点
信息复用，被计算过的区域下次计算会更快更准确
## 缺点
梯度变化缓慢，尤其是一个很亮的点被遮挡或在地平线上的情况，如下图所示：
![](论文阅读/Irradiance%20Caching%20and%20Radiance%20Caching/pics/9.png)
