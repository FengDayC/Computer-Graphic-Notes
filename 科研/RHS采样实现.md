# Path Tracing原始实现
## 一次光线弹射的框架
假设当前通量为$\beta$
一次光线弹射分为三个采样+停止条件判断，一次采样的总贡献如下：
$$
L=\omega_{L_e}L_e+\omega_{NEE}L_{light}
$$
假设一次迭代中，光路为$A\rightarrow B$，每次迭代在B点结算当前光路的贡献
## 自发光(相当于无方向采样)
考虑到自发光的光路是上一个交点到当前交点（光源），因此需要使用MIS来混合两种采样策略（当前光源的采样和上一个交点的BSDF采样）。则MIS权重公式如下：
$$
\omega_{L_e}=\frac{p_{BSDF_A}(A\rightarrow B)}{p_{BSDF_A}(A\rightarrow B)+p_e(B\rightarrow A)}
$$
## 直接光源采样
直接光源采样的光路是当前着色点到光源，因此采用MIS的权重为：
$$
\omega_{NEE}=\frac{p_{light}(Light\rightarrow B)}{p_{light}(Light\rightarrow B)+p_{BSDF_B}(B\rightarrow Light)}
$$
## 终止条件
使用俄罗斯轮盘赌终止，考虑每次都终止一部分光线，将停止概率设置为所有光线的最大通量，因此采用以下公式进行：
$$
\begin{cases}
\beta_{next} = \frac{\beta}{p_{rr}} & 以概率p_{rr}=min(0.95,max(所有光线的\beta))\\
终止路径 & 以概率(1-p_{rr})
\end{cases}
$$
# Guided RHS实现
Guided RHS考虑两个采样策略：一是直接光源采样，得到真实的Radiance结果；二是基于BSDF的采样，得到无方向的采样结果；三是根据SD树对光场的预测采样，得到引导的采样样本。根据PPG，第二第三种采样的pdf可以由一个参数$\alpha$来混合，但目前实现考虑混合无方向的BSDF采样和引导采样策略
## MIS权重
当前着色点$x$，入射方向$\omega_i$，考虑以上三种采样策略，会产生三个光路方向：$\omega_{BSDF}$、$\omega_{NEE}$、$\omega_{Guided}$
MIS权重分别如下：
$$
\begin{array}\\
w_{NEE}=\frac{pdf_{NEE}(\omega_{NEE})}{pdf_{NEE}(\omega_{NEE})+pdf_{Guided}(\omega_{NEE})+pdf_{BSDF}(\omega_{NEE})}\\
w_{BSDF}=\frac{pdf_{BSDF}(\omega_{BSDF})}{pdf_{NEE}(\omega_{BSDF})+pdf_{Guided}(\omega_{BSDF})+pdf_{BSDF}(\omega_{BSDF})}\\
w_{Guided}=\frac{pdf_{Guided}(\omega_{Guided})}{pdf_{NEE}(\omega_{Guided})+pdf_{Guided}(\omega_{Guided})+pdf_{BSDF}(\omega_{Guided})}\\
\end{array}
$$
## 采样过程
一次RHS采样的过程如下：
$$
\begin{array}\\
RHS(x,\omega_o)=L_e+L_{NEE}+L_{Guided}\\
\omega_{NEE},\omega_{Guided}=sample(x,\omega_o)\\
L_{NEE}=w_{NEE}\frac{R_{light}}{pdf_{NEE}(\omega_{NEE})}\\
L_{Guided}=w_{Guided}\frac{Model(x,\omega_{Guided})}{pdf_{Guided}(\omega_{Guided})}\\

\end{array}
$$