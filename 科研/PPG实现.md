# 原始SD树引导方法
## 框架
使用两棵SD树，第$k$次迭代使用$\hat{L}_{k-1}$记录的分布来引导样本分布，然后将新的Radiacne分布记录到$\hat{L}_k$中，然后丢弃$\hat{L}_{k-1}$，重复这个迭代过程。
## 分裂条件
每一次迭代正常插入，在迭代结束更新树的时候执行分裂
+ S树分裂条件：叶结点保存的顶点数超过一个设定值$c\cdot \sqrt{2^k}$时执行分裂，其中$k$时迭代次数
+ D树分裂条件：叶结点保存的flux超过总flux的一定比例时执行分裂，实际使用的是$1\%$
## D树结点值问题
D树的每个结点保存的是Radiant Intensity，也就是单位立体角上的光通量。
D树的每个结点维护的区域是一个立体角，因此x坐标为$cos\theta$值，y坐标为$\phi$值。有：
$$
dA=|cos\theta_{max}-cos\theta_{min}|\cdot|\phi_{max}-\phi_{min}|
$$
每当进来一个光子，radiance为$l$，结点的Radiant Intensity增加$l \cdot dA$，即认为这个光子覆盖了整个立体角区域