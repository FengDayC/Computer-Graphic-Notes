# Irradiance&Radiance Caching的基本思路
将场景的光照结果直接存储在一些特定的表面上，用数据结构维护。在计算其他点的光照结果时，复用这些点保存的光照信息来进行插值。

# Irradiance Caching(IC)
## 提出
+ 1988年的文章[[A ray tracing solution for diffuse interreflection]]提出了基础的Irradiance Caching。
+ 1992年的文章[[Irradiance Gradient]]，针对原IC算法中Irradiance Gradient的计算方法不足以精确计算复杂变化的情况，改进了梯度计算方法，提高了准确度。
+ 2006