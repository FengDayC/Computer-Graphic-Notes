# 数学依据
对于环境光照而言，着色点的位置与采样环境贴图的点无关，并且不用考虑自发光项，渲染方程可写为：
$$
L(\mathbf{x},\mathbf{\omega}_o)=\int_\Omega L(\mathbf{\omega}_i)V(\mathbf{x},\mathbf{\omega}_i)f(\mathbf{\omega}_o,\mathbf{\omega}_i)(\mathbf{\omega}_i\cdot\mathbf{n})d\omega_i
$$
环境光照项可以做基函数拆分：
$$
L(\omega_i)=\sum_j\alpha_jY_j(\mathbf{\omega}_i)
$$
则渲染方程可变为：
$$
L(\mathbf{x},\mathbf{\omega}_o)=\sum_j\alpha_j\int_\Omega Y_j(\mathbf{\omega}_i) V(\mathbf{x},\mathbf{\omega}_i)f(\mathbf{\omega}_o,\mathbf{\omega}_i)(\mathbf{\omega}_i\cdot\mathbf{n})d\omega_i
$$
## Lambertian Object
对于单纯的漫反射表面而言，有：
$$
f(\mathbf{\omega}_i,\mathbf{\omega}_o)=\frac{\rho}{\pi}
$$
因此渲染方程可写为：
$$
L(\mathbf{x},\mathbf{\omega}_o)=\sum_j\alpha_j\frac{\rho}{\pi}\int_\Omega Y_j(\mathbf{\omega}_i) V(\mathbf{x},\mathbf{\omega}_i)(\mathbf{\omega}_i\cdot\mathbf{n})d\omega_i
$$
将后面的积分项拆出来：
$$
T_j=\frac{\rho}{\pi}\int_\Omega Y_j(\mathbf{\omega}_i) V(\mathbf{x},\mathbf{\omega}_i)(\mathbf{\omega}_i\cdot\mathbf{n})d\omega_i
$$
整个渲染方程变为：
$$
L(\mathbf{x},\mathbf{\omega}_o)=\sum_j\alpha_jT_j
$$
可以写为向量形式：
$$
L(\mathbf{x},\mathbf{\omega}_o)=\mathbf{\alpha}\mathbf{T}
$$
在实际计算时，需要对每个着色点的$\mathbf{T}$进行预计算和保存，在实际着色时直接使用光照条件向量$\mathbf{\alpha}$与此向量进行点乘得到光照结果
## Glossy Object
对于光滑表面而言，BRDF不再是常数，因此渲染方程回到了原始形态，只能对光照项进行拆分：
$$
T_j(\mathbf{\omega}_o)=\int_\Omega Y_j(\mathbf{\omega}_i) V(\mathbf{x},\mathbf{\omega}_i)f(\mathbf{\omega}_o,\mathbf{\omega}_i)(\mathbf{\omega}_i\cdot\mathbf{n})d\omega_i
$$
由于$T(\mathbf{\omega}_o)$也是一个球面函数，因此也可以进行基函数拆分，有：
$$
T_j(\mathbf{\omega}_o)=\sum_k\beta_{jk}Y_k(\mathbf{\omega}_o)
$$
拆分之后，得到光滑表面的光照计算公式：
$$
L(\mathbf{x},\mathbf{\omega}_o)=\sum_j\alpha_j\sum_k\beta_{jk}Y_k(\mathbf{\omega}_o)
$$
事实上，这是一个矩阵乘法，写成矩阵的形式：
$$
\begin{matrix}
L(\mathbf{x},\mathbf{\omega}_o)=\mathbf{A}\mathbf{B}\mathbf{y}\\
\mathbf{A}=diag(\alpha_1,...\alpha_n)\\
\mathbf{B}=\{\beta_{jk}\}\\
\mathbf{y}=(Y_1(\mathbf{\omega}_o)....Y_k(\mathbf{\omega}_o)....)^T
\end{matrix}
$$
