# 摘要
提出了一种新的计算Radiance Gradient的方法，基于对半球面中心移动朝着切线和副切线方向移动对对采样区域的变化的分析。

# 算法描述
采样区域如下所示：
![](次表面散射/Irradiance%20Caching%20and%20Radiance%20Caching/pics/30.png)
计算出的在$\mathbf{u_k}与\mathbf{v_k}$方向的偏导数如下：
![](次表面散射/Irradiance%20Caching%20and%20Radiance%20Caching/pics/31.png)
