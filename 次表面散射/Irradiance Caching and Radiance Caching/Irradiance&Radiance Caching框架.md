# Irradiance&Radiance Caching的基本思路
将场景的光照结果直接存储在一些特定的表面上，用数据结构维护。在计算其他点的光照结果时，复用这些点保存的光照信息来进行插值。

# Irradiance Caching(IC)
## 发展
+ 1988年的文章[[A ray tracing solution for diffuse interreflection]]提出了基础的Irradiance Caching。
+ 1992年的文章[[Irradiance Gradient]]，针对原IC算法中Irradiance Gradient的计算方法不足以精确计算复杂变化的情况，引入了旋转梯度和平移梯度，改进了梯度计算方法，提高了准确度。
## 不足
Irradiance Caching由于其梯度不足以描述高频的Specular信息，因此会存在估计不准确的问题和漏光的问题
![](次表面散射/Irradiance%20Caching%20and%20Radiance%20Caching/pics/13.png)
![](次表面散射/Irradiance%20Caching%20and%20Radiance%20Caching/pics/14.png)

# Radience Caching(RC)
## 发展
+ 为了解决Irradience Caching中对于Glossy表面的估计不准确问题，2006年的[[Radiance Caching for Efficient Global Illumination Computation]]用球谐函数来表示入射Radiance和BRDF，实现了更加精确的Radiance Caching
+ 