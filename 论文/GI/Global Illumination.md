# 核心问题

# 神经方法
## 对象添加
### [[NeLT Object-Oriented Neural Light Transfer]]
思想：通过一个个添加对象，编码GI的变化函数，得到GI结果
限制：
+ 推理时间较长
+ 无法解决Deformable的物体
+ 对高频信息的还原有限
### [[Neural Global Illumination via Superposed Deformable Feature Fields]]
思想：基于NeLT，通过一个可形变的神经辐射场来拟合高频的场景信息从而实现对焦散、阴影等效果的更好实现
限制：
+ 推理时间较长
+ 

## 神经光探针
[[Neural Light Grid]]：直接用神经网络编码光照探针
