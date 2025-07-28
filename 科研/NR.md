$$
\begin{array}\\
L(x,\omega_o)=\underbrace{L_e(x,\omega_o)+\int_{\mathcal{H}^2}f(x,\omega_i,\omega_o)L_\theta(x',-\omega_i)(\omega_i\cdot \mathcal{n})d\omega_i}_{\text{Right Hand Side}}\\
\approx L_e(x,\omega_o)+\frac{\int_{\mathcal{H}^2}L_\theta(x',-\omega_i)d\omega_i}{\int_{\mathcal{H}^2}d\omega_i}\int_{\mathcal{H}^2}f(x,\omega_i,\omega_o)(\omega_i\cdot \mathcal{n})d\omega_i\\
\end{array}
$$
# 单一模型进行预测与引导
## 问题1
路径引导如何将预测的radiance转换成为pdf？
### 方法1
可以让模型预测Radiance的同时预测一个pdf(双输出)
改进：这个pdf可以采用GMM、vMF等方法来规范化
### 暴力方法
可以多做一次RHS得到归一化常数。
可以绕过pdf，做多次采样选择最大radiance方向。
## 问题2
如何解决动态场景的问题？