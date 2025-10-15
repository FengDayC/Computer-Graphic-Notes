# 结论
## 发现
+ 探针信息保存在FPrecomputedVolumetricLightmapData结构体(文件:Engine\Source\Runtime\Engine\Public\PrecomputedVolumetricLightmap.h)里面，通过FPrecomputedVolumetricLightmap::AddToScene(文件：Engine\Source\Runtime\Engine\Private\PrecomputedVolumetricLightmap.cpp)加载到场景中，调用堆栈如下：
![](科研/神经探针/pics/6.png)
+ 这个探针数据是从关卡的BuiltData加载来的(可能可以直接读取这个文件取得)
+ Shader代码中直接插值附近探针的球谐向量并且拿来恢复Diffuse，所以会漏光
## FPrecomputedVolumetricLightmapData结构
一个场景中的实际数据：
![](科研/神经探针/pics/4.png)

### 大致可以确定含义的一些参数
+ Bounds：包围盒
+ BrickDataDimensions：探针的Extent
+ BrickData:
	+ AmbientVector：探针的0阶SH系数的数据，绑定纹理后的xyz通道分别保存RGB三个通道的0阶SH系数
	+ SHCoefficients：是一个数据数组，一共有六个数据，对应六张纹理
		+ 下标为0,2,4的数据：xyz对应RGB通道的1阶SH系数，w对应RGB通道的2阶SH系数的第5个
		+ 下标为1,3,5的数据：xyzw对应RGB通道的2阶SH系数的前4个
### 还不清楚的一些参数
+ bTransient
+ IndirectionTexture和IndirectionTextureDimensions和IndirectionOriginal:似乎是间接纹理相关的索引
+ BrickData.SkyBentNormal和BrickData.DirectionalLightShadowing:似乎和光源的计算相关
+ BrickSize:一个int变量，不清楚具体指什么

## 探针索引方式

探针信息（球谐系数+用来计算环境光和平行光的信息）保存在体纹理中，在渲染的时候直接使用世界坐标计算得到的UVW去采样这个体纹理，世界坐标向采样坐标转换的代码(文件: Engine\Shaders\Private\VolumetricLightmapShared.ush)如下：

```c++
float3 ComputeVolumetricLightmapBrickTextureUVs(float3 WorldPosition) // RT_LWC_TODO
{
	// Compute indirection UVs from world position
	float3 IndirectionVolumeUVs = clamp(WorldPosition * View.VolumetricLightmapWorldToUVScale + View.VolumetricLightmapWorldToUVAdd, 0.0f, .99f);//变换世界坐标到[0,1]^3中(标准空间)
	float3 IndirectionTextureTexelCoordinate = IndirectionVolumeUVs * View.VolumetricLightmapIndirectionTextureSize;//得到间接纹理上纹素坐标(间接纹理空间)
	float4 BrickOffsetAndSize = View.VolumetricLightmapIndirectionTexture.Load(int4(IndirectionTextureTexelCoordinate, 0));//这里有点没看懂，似乎是用间接纹理坐标去整个大纹理中采样，需要加上的偏移量

	float PaddedBrickSize = View.VolumetricLightmapBrickSize + 1;
	return (BrickOffsetAndSize.xyz * PaddedBrickSize + frac(IndirectionTextureTexelCoordinate / BrickOffsetAndSize.w) * View.VolumetricLightmapBrickSize + .5f) * View.VolumetricLightmapBrickTexelSize;
}
```
与CPU代码对应，可以得到一些参数含义：
+ VolumetricLightmapWorldToUVScale:包围盒大小的倒数
+ VolumetricLightmapWorldToUVAdd:包围盒左下坐标\*包围盒大小倒数
+ VolumetricLightmapIndirectionTextureSize:间接纹理的三个维度，在我的场景中是(16,16,16)
+ VolumetricLightmapBrickSize:不清楚
+ VolumetricLightmapBrickTexelSize:BrickDataDimensions的倒数



# Shader寻找过程

在Engine\Shaders\Private\VolumetricLightmapShared.ush中，有获取FThreeBandSHVectorRGB的函数GetVolumetricLightmapSH3，这个函数返回了一个球谐向量(rgb每个通道一共有4+4+1=9个球谐系数，**初步判断使用了012三层球谐函数**):
![](科研/神经探针/pics/2.png)
从这个函数出发开始寻找
## 向下层寻找
在Engine\Shaders\Private\BasePassPixelShader.usf的GetPrecomputedIndirectLightingAndSkyLight函数中，调用了GetVolumetricLightmapSH3，得到的球谐向量与计算好的球谐基函数点乘，得到了恢复好的漫反射光照OutDiffuseLighting(应该是辐照度):
![](科研/神经探针/pics/3.png)
这段Shader代码应该对应的BasePass的片元着色器，是用来上色的

### 球谐函数的阶数
进入这个DotSH3函数，可以看到只是简单地对两个9维的向量点乘，代码如下：
```c++
half DotSH3(FThreeBandSHVector A,FThreeBandSHVector B)
{
	half Result = dot(A.V0, B.V0);
	Result += dot(A.V1, B.V1);
	Result += A.V2 * B.V2;
	return Result;
}
```
进入这个CalcDiffuseTransferSH3，并且查看生成球谐基在法线方向的值的SHBasisFunction3函数，与球谐函数表对照，可知：
```c++
FThreeBandSHVector SHBasisFunction3(half3 InputVector)
{
	FThreeBandSHVector Result;
	// These are derived from simplifying SHBasisFunction in C++
	Result.V0.x = 0.282095f; 
	Result.V0.y = -0.488603f * InputVector.y;
	Result.V0.z = 0.488603f * InputVector.z;
	Result.V0.w = -0.488603f * InputVector.x;

	half3 VectorSquared = InputVector * InputVector;
	Result.V1.x = 1.092548f * InputVector.x * InputVector.y;
	Result.V1.y = -1.092548f * InputVector.y * InputVector.z;
	Result.V1.z = 0.315392f * (3.0f * VectorSquared.z - 1.0f);
	Result.V1.w = -1.092548f * InputVector.x * InputVector.z;
	Result.V2 = 0.546274f * (VectorSquared.x - VectorSquared.y);

	return Result;
}
```
V0.x:第0层球谐基函数(1个)
V0.yzw:第一层球谐基函数(3个)
V1.xyzw+V2:第二层球谐基函数(5个)
### 采样体数据的坐标
找到调用GetVolumetricLightmapSH3的代码：
```c++
FThreeBandSHVectorRGB IrradianceSH = GetVolumetricLightmapSH3(VolumetricLightmapBrickTextureUVs);
```
回到上一层找到VolumetricLightmapBrickTextureUVs的来源：
```c++
#if PRECOMPUTED_IRRADIANCE_VOLUME_LIGHTING
	VolumetricLightmapBrickTextureUVs = ComputeVolumetricLightmapBrickTextureUVs(WSHackToFloat(GetWorldPosition(MaterialParameters)));
#endif
```
进入ComputeVolumetricLightmapBrickTextureUVs：
```c++
float3 ComputeVolumetricLightmapBrickTextureUVs(float3 WorldPosition) // RT_LWC_TODO
{
	// Compute indirection UVs from world position
	float3 IndirectionVolumeUVs = clamp(WorldPosition * View.VolumetricLightmapWorldToUVScale + View.VolumetricLightmapWorldToUVAdd, 0.0f, .99f);//变换世界坐标到[0,1]^3中(标准空间)
	float3 IndirectionTextureTexelCoordinate = IndirectionVolumeUVs * View.VolumetricLightmapIndirectionTextureSize;//得到间接纹理上纹素坐标
	float4 BrickOffsetAndSize = View.VolumetricLightmapIndirectionTexture.Load(int4(IndirectionTextureTexelCoordinate, 0));

	float PaddedBrickSize = View.VolumetricLightmapBrickSize + 1;
	return (BrickOffsetAndSize.xyz * PaddedBrickSize + frac(IndirectionTextureTexelCoordinate / BrickOffsetAndSize.w) * View.VolumetricLightmapBrickSize + .5f) * View.VolumetricLightmapBrickTexelSize;
}
```
这里的关键参数如下：
+ View.VolumetricLightmapWorldToUVScale：体数据覆盖空间大小的倒数
+ View.VolumetricLightmapWorldToUVAdd：体数据起始位置的标准坐标
+ View.VolumetricLightmapIndirectionTextureSize
+ View.VolumetricLightmapIndirectionTexture
这些可能在CPU端被设置
## 向上层寻找
进入GetVolumetricLightmapSHCoefficients0函数，如下：
```c++
void GetVolumetricLightmapSHCoefficients0(float3 BrickTextureUVs, out float3 AmbientVector, out float4 SHCoefficients0Red, out float4 SHCoefficients0Green, out float4 SHCoefficients0Blue)
{
	AmbientVector = GetVolumetricLightmapAmbient(BrickTextureUVs);
	SHCoefficients0Red = Texture3DSampleLevel(View.VolumetricLightmapBrickSHCoefficients0, PIVSharedSampler0, BrickTextureUVs, 0) * 2 - 1;
	SHCoefficients0Green = Texture3DSampleLevel(View.VolumetricLightmapBrickSHCoefficients2, PIVSharedSampler2, BrickTextureUVs, 0) * 2 - 1;
	SHCoefficients0Blue = Texture3DSampleLevel(View.VolumetricLightmapBrickSHCoefficients4, PIVSharedSampler4, BrickTextureUVs, 0) * 2 - 1;

	// Undo normalization done in FIrradianceBrickData::SetFromVolumeLightingSample
	float4 SHDenormalizationScales0 = float4(
		0.488603f / 0.282095f, 
		0.488603f / 0.282095f, 
		0.488603f / 0.282095f, 
		1.092548f / 0.282095f);

	SHCoefficients0Red = SHCoefficients0Red * AmbientVector.x * SHDenormalizationScales0;
	SHCoefficients0Green = SHCoefficients0Green * AmbientVector.y * SHDenormalizationScales0;
	SHCoefficients0Blue = SHCoefficients0Blue * AmbientVector.z * SHDenormalizationScales0;
}
```
可以看到每个三个通道分别采样了VolumetricLightmapBrickSHCoefficients0、VolumetricLightmapBrickSHCoefficients2、VolumetricLightmapBrickSHCoefficients4这三张纹理，并且AmbientVector是从VolumetricLightmapBrickAmbientVector这张纹理中得到的。
回到看到这段代码：
```c++
	FThreeBandSHVectorRGB IrradianceSH;
	// Construct the SH environment
	IrradianceSH.R.V0 = float4(AmbientVector.x, SHCoefficients0Red.xyz);
	IrradianceSH.R.V1 = float4(SHCoefficients0Red.w, SHCoefficients1Red.xyz);
	IrradianceSH.R.V2 = SHCoefficients1Red.w;

	IrradianceSH.G.V0 = float4(AmbientVector.y, SHCoefficients0Green.xyz);
	IrradianceSH.G.V1 = float4(SHCoefficients0Green.w, SHCoefficients1Green.xyz);
	IrradianceSH.G.V2 = SHCoefficients1Green.w;

	IrradianceSH.B.V0 = float4(AmbientVector.z, SHCoefficients0Blue.xyz);
	IrradianceSH.B.V1 = float4(SHCoefficients0Blue.w, SHCoefficients1Blue.xyz);
	IrradianceSH.B.V2 = SHCoefficients1Blue.w;
```
和上面寻找球谐基函数的对应
## 结论
+ Shader中直接使用世界位置插值探针的球谐向量，没有针对特定的着色点做是否对探针可见的判断，因此漏光
+ Shader中用两张四通道纹理+一张单通道纹理
# CPU代码寻找过程
## 设置球谐函数的代码
+ 在SceneRendering.cpp里面的SetupUniformBufferParameters函数中，UE设置了Shader用到的Uniform变量（名称相同）
+ 其中调用了SetupPrecomputedVolumetricLightmapUniformBufferParameters方法，里面对VLM的参数进行了设置
+ 在断点中，看到有一个类型为FPrecomputedVolumetricLightmapData\*的VolumetricLightmapData变量
+ 设置球谐向量的体纹理的代码如下，结合前面Shader的代码，这些球谐向量的体纹理对应的应该是：
```c++
//间接纹理
ViewUniformShaderParameters.VolumetricLightmapIndirectionTexture = OrBlack3DUintIfNull(VolumetricLightmapData->IndirectionTexture.Texture);
//三通道纹理，xyz分别对应R,G,B通道的0阶SH系数
ViewUniformShaderParameters.VolumetricLightmapBrickAmbientVector = OrBlack3DIfNull(BrickData->AmbientVector.Texture);
//四通道纹理，xyz对应R通道的1阶SH系数，w对应R通道的2阶SH系数的第5个
ViewUniformShaderParameters.VolumetricLightmapBrickSHCoefficients0 = OrBlack3DIfNull(BrickData->SHCoefficients[0].Texture);
//四通道纹理，xyzw对应R通道的2阶SH系数的前4个
ViewUniformShaderParameters.VolumetricLightmapBrickSHCoefficients1 = OrBlack3DIfNull(BrickData->SHCoefficients[1].Texture);
//G和B通道以此类推
ViewUniformShaderParameters.VolumetricLightmapBrickSHCoefficients2 = OrBlack3DIfNull(BrickData->SHCoefficients[2].Texture);
ViewUniformShaderParameters.VolumetricLightmapBrickSHCoefficients3 = OrBlack3DIfNull(BrickData->SHCoefficients[3].Texture);
ViewUniformShaderParameters.VolumetricLightmapBrickSHCoefficients4 = OrBlack3DIfNull(BrickData->SHCoefficients[4].Texture);
ViewUniformShaderParameters.VolumetricLightmapBrickSHCoefficients5 = OrBlack3DIfNull(BrickData->SHCoefficients[5].Texture);
//应该是用来计算环境光的
ViewUniformShaderParameters.SkyBentNormalBrickTexture = OrBlack3DIfNull(BrickData->SkyBentNormal.Texture);
//应该是用来计算平行光的
ViewUniformShaderParameters.DirectionalLightShadowingBrickTexture = OrBlack3DIfNull(BrickData->DirectionalLightShadowing.Texture);
```
+ 除了设置球谐向量的纹理之外，还有一些设置世界坐标到用于采样体数据的uvw坐标的变换的量，如下：
```c++
//整个体积大小的倒数
ViewUniformShaderParameters.VolumetricLightmapWorldToUVScale = (FVector3f)InvVolumeSize;
//体积的左下坐标在标准坐标系中的uvw
ViewUniformShaderParameters.VolumetricLightmapWorldToUVAdd = FVector3f(-VolumeBounds.Min * InvVolumeSize);
//间接纹理的大小
ViewUniformShaderParameters.VolumetricLightmapIndirectionTextureSize = FVector3f(VolumetricLightmapData->IndirectionTextureDimensions);
//探针在XYZ方向上数量
ViewUniformShaderParameters.VolumetricLightmapBrickSize = VolumetricLightmapData->BrickSize;
//数量的倒数
ViewUniformShaderParameters.VolumetricLightmapBrickTexelSize = (FVector3f)InvBrickDimensions;
```
## 体数据的保存位置
根据断点结果，主要的体数据纹理保存在两个变量下面：
### Scene->VolumetricLightmapSceneData
从这段代码出发:
```c++
const FPrecomputedVolumetricLightmapData* VolumetricLightmapData = Scene->VolumetricLightmapSceneData.GetLevelVolumetricLightmap()->Data;
```
往上找可以知道Scene类的VolumetricLightmapSceneData在初始化的时候被设置，具体的数据是通过FVolumetricLightmapSceneData类的AddLevelVolume函数赋值的。**通过断点，这个函数只有在Build场景的光照才被调用**。因此可以确定是设置烘焙好的VLM参数的函数。
并且，调用AddLevelVolume的上一级函数是FScene::AddPrecomputedVolumetricLightmap

### GVolumetricLightmapBrickAtlas
这个变量定义在Engine\Source\Runtime\Engine\Private\PrecomputedVolumetricLightmap.cpp中，但是没有找到对其进行赋值的代码。
对其数据类型FVolumetricLightmapBrickAtlas的Insert函数进行断点，发现在PrecomputedVolumetricLightmap.cpp的SetData中对其进行了调用，其参数NewData保存了CPU端的体数据
![](科研/神经探针/pics/4.png)
AddToScene的上一级函数是SetData函数，一路向上找可以发现从场景BuiltData中得到的探针信息