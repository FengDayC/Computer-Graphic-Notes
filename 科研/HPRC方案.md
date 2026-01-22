# Featuremap方案
## 简单
直接使用Featuremap+MLP架构
### 官方方案 - [ x ] 
一套光图一套Featuremap，时间信息由MLP处理
### 共同特征 - [ x ] 
一套Featuremap+多套时间相关Featuremap+MLP
安全步数：100000步
推理速度的影响：
+ MLP大小产生主要影响
+ 网格数量产生次要影响
featuremap大小影响重建质量

## 自编码器结构 - [   ] 
使用自编码器训练Featuremap

## 基函数 - [   ] 
用神经网络分离投影到基函数上的几何项和光照项，最终结果点乘

## 多网格 - [   ] 
uv网格+ut网格+vt网格+MLP
baseline:
+ 原分辨率+4层256MLP
+ 原分辨率(16+16)+4层64MLP
较好：分辨率缩小2倍+4层64MLP(42分)
思路：大网格+小MLP，通过量化降低网格实际大小

# 量化
无法在pytorch下进行量化推理

# 编码
## 频率编码
# MLP数据保存
