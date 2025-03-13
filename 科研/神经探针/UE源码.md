+ 在SceneRendering.cpp里面的SetupUniformBufferParameters函数中，UE设置了Shader用到的Uniform变量
+ 其中调用了SetupPrecomputedVolumetricLightmapUniformBufferParameters方法，里面对VLM的参数进行了设置
+ 对应的VLM数据结构代码如下：
```c++
const FPrecomputedVolumetricLightmapData* VolumetricLightmapData = Scene->VolumetricLightmapSceneData.GetLevelVolumetricLightmap()->Data;
```
