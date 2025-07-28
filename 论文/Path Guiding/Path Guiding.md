# PG的核心思想
将有限的采样资源投入到产生高质量结果的路径上。具体来说，在采样过程中，从先前的采样中学习场景信息，并利用这些信息优化之后的采样。
# PG总述
## 直接光照采样
直接光照采样通常有两种方式，一种是对采样点的各个方向进行采样(unidirectional sampling)，一种是直接对光源进行采样(next-event estimation)。
对于unidirectional sampling，PG会学习光源的方向分布，并将其储存在场景中。这样的好处是可以很好的处理光源遮挡问题，但在光源少、遮挡少的场景下效率不如NNE。
对于NEE，首先要考虑的问题是多光源下的遮挡问题，这个问题通常由将场景进行分割以指导光源选择方法解决。其次，光源每一点的采样数目也是一个重要问题，对于昏暗的场景，可以减少采样数目来节省时间。
## 间接光照采样
间接光照采样最重要的问题在于如何进行高效的方向采样。这方面有很多的工作:
+ [[Product Importance Sampling for Light Transport Path Guiding]]
+ [[Zero-Variance Based Sampling for Volume Path Guiding]]
+ [[Practical Path Guiding for Efficient Light-Transport Simulation]]
+ [[On-line Learning of Parametric Mixture Models for Light Transport Simulation]]
+ [[Selective guided sampling with complete light transport paths]]
+ [[Learning Light Transport the Reinforced Way]]:强化学习
+ [[Neural Importance Sampling]]:神经网络
## 聚焦/焦散
对于一些特殊的焦散(如水滴、角色眼睛等折射情况，也被称为Specular-Diffuse-Specular型)，可以通过[[Manifold Next Event Estimation]]处理，除此之外，也可以通过光子映射处理[[Light transport simulation with vertex connection and merging]]，但需要高密度的光子。
理论上来说，焦散可以通过普通的Path Guiding来处理，但会花费相当多的计算资源。方向引导的Path Guiding可以渲染焦散，但会出现闪烁的噪点，而这样的噪点可以通过clamp来去噪。
## Glossy表面
对于PG来说，要产生高质量的采样，就需要考虑光照和BSDF的乘积，尤其是Glossy的情况。如果只考虑光照而不考虑BSDF，则更适用于Diffuse的情况。
要混合使用这两种情况，一种最简单的方案是找一个roughness阈值，分别处理高于和低于阈值的情况。一种方案是使用采样重要性重采样的方法(SIR)[[# Sequential monte carlo methods for physically based rendering]]。另一种方案是对场景的两种分布进行学习：一种是学习乘积，指导主要的光线方向；另一种用于指导其余潜在的光线弹射。
## 引导俄罗斯轮盘赌和分裂算法
路径追踪中一个重要的问题在于何时停止追踪，通常使用的是基于albedo的RR和Splitting来控制。这种方法存在一些问题：首先是光路过早结束，导致噪声。其次是在高albedo的场景下，会花费非常多的计算时间来计算反射。
通过全局信息引导的俄罗斯轮盘赌和分裂可以解决这一问题[[Adjoint-Driven Russian Roulette and Splitting in Light Transport Simulation]]
## 引导光子映射
在大场景(如被环境贴图和平行光这样的远光源照亮的室外场景)下，光子和双向路径追踪渲染器的效率会变得非常低，主要是因为采样到重要光路的概率很低。
## 引导双向路径追踪算法

# 多光渲染中的贝叶斯推理
[[Bayesian online regression for adaptive illumination sampling]]
# Practical Path Guiding(PPG)
[[Practical Path Guiding for Efficient Light-Transport Simulation]]
# 零方差路径引导
[[Volume Path Guiding based on Zero-Variance Random Walk Theory]]

# 神经方法
[[Efficient Neural Path Guiding with 4D Modeling]]
