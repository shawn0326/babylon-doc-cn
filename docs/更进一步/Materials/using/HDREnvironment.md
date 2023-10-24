# 使用 HDR 环境进行 PBR

## 介绍

强烈推荐的设置环境纹理的方法是通过 HDR 就绪文件（DDS 或 ENV），其中包含带有预过滤 MipMap 的立方体纹理。

要加载 HDR 环境图，您可以使用 [createDefaultEnvironment](https://doc.babylonjs.com/typedoc/classes/BABYLON.Scene#createDefaultEnvironment)：

````javascript
scene.createDefaultEnvironment();
````

这将从assets.babylonjs.com加载文件[environmentSpecular.env](https://assets.babylonjs.com/environments/environmentSpecular.env)。

要加载自定义环境纹理，只需设置 scene.environmentTexture：

````javascript
const hdrTexture = BABYLON.CubeTexture.CreateFromPrefilteredData("textures/environment.env", scene);
scene.environmentTexture = hdrTexture;
````

您还可以使用 createDefaultEnvironment() 中的选项：

````javascript
scene.createDefaultEnvironment({
    environmentTexture: "texture-url.env"
});
````

我们在下面详细介绍了创建此类文件的两种受支持的方法。

从 4.2 开始，我们现在支持直接在沙盒中进行预过滤！

.hdr 文件很容易在网络上找到，因此它看起来是最方便的过滤输入。

## 使用沙盒创建压缩环境纹理图

由于生成的 DDS 文件可能相对较大（512px 宽的文件为 32Mb），我们在 Babylon 中引入了一种特殊的方法来打包纹理。以下是创建 Babylon.js 中使用的 .env 文件的步骤：

* 去[sandbox](https://sandbox.babylonjs.com/)
* 拖放 PBR 场景文件（[示例](https://models.babylonjs.com/PBR_Spheres.glb)）
* 拖放您的 .dds 环境纹理文件（[示例](https://playground.babylonjs.com/textures/environment.dds)）
* 打开检查器，转到“工具”，然后单击“Generate .env texture”

![](https://doc.babylonjs.com/img/how_to/Environment/inspector-generate-env-texture.png)

您现在可以使用以下代码下载并使用您的 .env 环境图：

````javascript
scene.environmentTexture = new BABYLON.CubeTexture("environment.env", scene);
````

请参阅本页底部的 《什么是 .env（技术深度探索）》部分以了解更多信息。

## IBL 纹理工具

如果您有 .hdr 纹理，您可以使用 [IBL 纹理工具](https://www.babylonjs.com/tools/ibl/)将其轻松转换为 .env。

只需拖放您的 .hdr 文件，稍等片刻，然后将 .env 保存到您想要的任何位置。

## 直接使用.hdr文件

如果您想直接使用 .hdr 文件并且无法从沙箱或外部工具将其预过滤为 .env 或 .dds，您可以在加载纹理时执行此操作。

````javascript
const reflectionTexture = new BABYLON.HDRCubeTexture("./textures/environment.hdr", scene, 128, false, true, false, true);
````

由于预过滤是即时实现的，因此该方法在纹理加载方面会产生少量延迟。因此，最好使用 .env 或 .dds 文件以获得最佳性能。请注意，动态预过滤需要 WebGL2。

有时您甚至可能想要完全实时过滤（例如动画反射），您可能需要查看[反射探针教程](https://doc.babylonjs.com/features/featuresDeepDive/environment/reflectionProbes)。

## 外部工具

第一个工具依赖于名为 IBL Baker 的开源框架，而第二个工具则基于名为 Lys 的专有软件来创建更高分辨率的结果。

请注意，如果需要，您可以旋转环境纹理：

````javascript
const hdrRotation = 10; // in degrees
hdrTexture.setReflectionTextureMatrix(BABYLON.Matrix.RotationY(BABYLON.Tools.ToRadians(hdrRotation)));
````

### 从 IBL Baker 创建 dds 环境文件

您可以在以下地址找到 IBLBaker：https://github.com/derkreature/IBLBaker

克隆存储库后，您将能够进入到 /bin64 文件夹并启动 IBLBaker.exe。

现在使用“Load environment”按钮加载 HDR 图像（也可以从那里尝试一个）

我们建议使用环境比例来定义您想要的亮度。

您可能还想通过将镜面反射分辨率设置为 128 来减小输出大小，以确保模糊衰减的正确性：

![](https://doc.babylonjs.com/img/how_to/Environment/IBLbaker_DefaultSettings.png)

一旦您对整体结果感到满意，只需单击“保存环境”按钮即可开始！要选择的文件是 SpecularHDR 文件。

**请不要忘记写下全名和扩展名，以便正确保存。**

### 从 LYS 创建 dds 环境文件

[Lys](https://www.knaldtech.com/lys/) 可以在 [knaldtech](https://www.knaldtech.com/) 网站上找到。

使用 Lys，生成的 mipmap 的输出质量将达到更高标准，在粗糙度响应方面非常接近 Unity 标准材质。您可以使用 Lys 生成：128、256 或 512 像素宽的 dds 立方体纹理。

请按照以下步骤操作，以确保响应的物理正确性。

首先，您需要在主窗口中选择以下设置（根据您的选择调整大小128、256或512）：

![](https://doc.babylonjs.com/img/how_to/Environment/Lys_DefaultSettings_Main.png)

完成后，在首选项选项卡中，请设置旧版 dds 模式（Babylon.js 不支持 4CC）：

![](https://doc.babylonjs.com/img/how_to/Environment/Lys_DefaultSettings_Prefs.png)

在导出窗口中，您可以选择适当的选项（应最后将 DDS 设置为 32 位，因为我们已经看到它默认返回 8 位）：

![](https://doc.babylonjs.com/img/how_to/Environment/Lys_DefaultSettings_Export.png)

最后，您可以通过主选项卡导出纹理：

您已准备就绪，可以在 CubeTexture.CreateFromPrefilteredData 函数中使用导出的纹理。

## 使用纯立方体纹理

虽然使用 .dds 或 .env 立方体纹理是最佳选择，但您可能仍希望依赖经典立方体纹理（主要是出于尺寸原因）。所以，你仍然可以这样做：

````javascript
scene.environmentTexture = new BABYLON.CubeTexture("textures/TropicalSunnyDay", scene);
````

在这种情况下，您将无法获得 HDR 渲染，并且可能会出现一些视觉伪影（主要是在使用光泽度或粗糙度时）。

## 什么是 .env（技术深度探究）

我们要解决的 .env 问题是 IBL 环境纹理的大小和质量。我们决定实施自定义打包以简化这些资源的共享和下载。该文件需要跨平台工作以便于部署，因此我们不直接依赖压缩纹理。

然后，我们将 json 清单标头、多项式信息以及来自预过滤立方体纹理的 mipmap 链的所有面打包到一个文件（类似于 DDS 或 KTX）中，格式为 .png（压缩效果非常好，解码速度非常快）所有浏览器。）。

为了保持 png 的 HDR 支持，我们选择依赖 RGBD，因为它通过保持 [0-1] 范围不变（知道它通常使用更频繁）来提供比 RGBM 更好的低范围值分布。在需要时，运行时解码也比 LogLUV 更简单。这对我们来说似乎是最好的权衡。

RGBD 还在较低范围内提供无或低透明度，防止浏览器依赖 alpha 预乘来丢失最可见区域中的数据。我们还引入了与最大范围的特殊偏移，确保我们不会在旧版浏览器中达到有问题的 alpha 值（当 alpha 距离 0 太近时，颜色量化会产生不可接受的条带伪影）。

在 WebGL2 浏览器中，如果支持的话，我们会将数据解压为 HalfFloat 或 FullFloat 纹理，以加快运行时间并允许纠正一些插值。

该文件还打包了多项式谐波与球面多项式，以匹配 Babylon 内部的预期，无需再计算或转换它们，从而加快加载时间。

由于渲染到 LOD 甚至复制到半/全浮点纹理的 LOD 在基于 WebGL1 的浏览器上无法一致工作，因此我们在片段着色器中实时解包数据。由于 RGBD 插值不正确，我们通过不同的测试用例确保生成的视觉伪像值得传输增益。在我们测试过的纹理集中看起来还不错。

作为结果示例，我们现在可以依赖 512px 立方体大小的纹理，数据量约为 3Mb，而未打包版本的数据量为 32Mb，而不会注意到任何块质量下降。由于不再需要计算多项式，这也加快了我们到达第一帧的时间。









