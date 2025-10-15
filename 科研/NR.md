$$
\begin{array}\\
L(x,\omega_o)=\underbrace{L_e(x,\omega_o)+\int_{\mathcal{H}^2}f(x,\omega_i,\omega_o)L_\theta(x',-\omega_i)(\omega_i\cdot \mathcal{n})d\omega_i}_{\text{Right Hand Side}}\\
\approx L_e(x,\omega_o)+\frac{\int_{\mathcal{H}^2}L_\theta(x',-\omega_i)d\omega_i}{\int_{\mathcal{H}^2}d\omega_i}\int_{\mathcal{H}^2}f(x,\omega_i,\omega_o)(\omega_i\cdot \mathcal{n})d\omega_i\\
\end{array}
$$
# NPM
定义一个模型$NPM(x;\hat\Theta)=v1,v2,v3,...$，其中$\hat\Theta$是模型参数，$v1(\lambda_1,\mu_1,\kappa_1),v2(\lambda_2,\mu_2,\kappa_2),v3,...$等都是vMF的参数，每个v定义了一个vMF，多个vMF根据权重$\lambda$来进行混合得到一个pdf，这个pdf是目标分布$\mathcal{D}(\omega_{j})$的近似

现在考虑优化这一模型，对于一个指定位置$x$，指定方向$\omega_j$，目标分布值无法直接得到，但是可以得到此方向的光强$L(\omega_j)\propto \mathcal{D}(\omega_{j})$，
计算KL散度，$D_{\mathrm{KL}}(\mathcal{D} \| \mathcal{V} ; \Theta) \approx \frac{1}{N} \sum_{j=1}^{N} \frac{\mathcal{D}\left(\omega_{j}\right)}{\tilde{p}(\omega_{j})} \log \frac{\mathcal{D}\left(\omega_{j}\right)}{\mathcal{V}\left(\omega_{j} \mid \hat{\Theta}\right)}$,其中$\tilde{p}(\omega_{j})$是蒙特卡洛采样pdf
对这个KL散度计算梯度：$\nabla_{\hat{\Theta}} D_{\mathrm{KL}} \approx - \frac{1}{N} \sum_{j=1}^{N} \frac{\mathcal{D}(\omega_{j})\nabla_{\hat{\Theta}} \mathcal{V}(\omega_{j} \mid \hat{\Theta})}{\tilde{p}(\omega_{j})\mathcal{V}(\omega_{j} \mid \hat{\Theta})} \propto - \frac{1}{N} \sum_{j=1}^{N} \frac{L(\omega_{j})\nabla_{\hat{\Theta}} \mathcal{V}(\omega_{j} \mid \hat{\Theta})}{\tilde{p}(\omega_{j})\mathcal{V}(\omega_{j} \mid \hat{\Theta})}$
有了这个梯度，每次我可以求多个样本$L(x,\omega)$，我的模型优化思路应该是怎样的

# 尝试
+ 采用样本权重
+ 反向topk