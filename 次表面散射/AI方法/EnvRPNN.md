# 网络输入
每一层mip:
+ 0:密度
+ 1:零方差距离
+ 2-4:albedo
+ 5-7:irradiance
+ 8:g

# 验证清单
- [x] 模板数据组织顺序：通过对应特征位置输出对应数字验证
- [x] (0.5,0.5)中心点的采样坐标
- [ ] 中心点的position和normal：存在一些差距
	- [x] 计算的uv对不上GBuffer Texture的UV
	- [x] 输出两边的MVP矩阵
	- [x] postion和normal Texture插值设置不同造成的偏差
	- [x] 输出相同情况下的采样坐标
	- [x] 输出相同情况下的位置
- [x] 让所有的dir = forward:
- [x] tangent和bitangent的错误
- [x] 采样点uvw的错误