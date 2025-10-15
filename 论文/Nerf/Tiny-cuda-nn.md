# 输入编码
在类nerf的神经网络中，为了捕获高频细节，即随着输入急剧变化的信息，需要将平缓变化的位置输入进行剧烈变动的映射，例如将其位置坐标输入高频三角函数中，让网络的输入向量可以随着原始输入的缓慢改变而剧烈变化。由此，需要进行位置编码。
位置编码首先将位置进行归一化，然后再输入位置编码器，tiny-cuda-nn中，提供的位置编码器有以下几种：
## Frequancy Encoding
即nerf中的编码方式，公式为：
$$
enc(x)=(sin(2^0x),cos(2^0x),...,sin(2^(L-1)x),cos(2^(L-1)x))
$$
相当于做了一个傅里叶变换

## One-blob Encoding
one-blob encoding在输入空间中设置一些激活区域，使得靠近这些激活区域的输入编码为较大的值，同时其他区域的输入编码为较小的值，并且一般采用Gaussian核进行这个操作
$$
\begin{array}\\
enc(x)=(enc_0(x),...enc_n(x))\\
enc_i(x)=exp(-\frac{x-\mu_i}{2\sigma^2})
\end{array}
$$
在tiny-cuda-nn中，这个$\mu$是默认均匀在输入空间中的

## Trianglewave Encoding
由于计算三角函数和高斯函数较为复杂，因此以上两种编码方式的“核函数”可以简化为简单的多项式函数。NRC[[Neural Radiance Caching]]中的做法是：
![](1.png)
即实际计算中的$sin,cos,gaussian$用以上多项式函数来做一个近似，从而优化计算效率

## Hash Encoding
即Instant NGP[[Instant Neural Graphics Primitives with a Multiresolution Hash Encoding]]，是一种隐性的编码方式，编码得到的值通过学习得到保存在多分辨率网格中，每次通过采样网格得到编码结果。

## SH Encoding
即对于给定的输入，将其指定阶的球谐函数值拼接起来作为输入

## Identity Encoding
恒等编码，没什么特别的


