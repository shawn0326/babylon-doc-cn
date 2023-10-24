# 基于物理的渲染简介

## 介绍

基于物理的渲染 (PBR) 的目的是模拟现实生活中的照明。

PBR 是一组技术；它不会强迫您特别选择其中之一。其中，我们可以举一些例子：

* [迪士尼](https://blog.selfshadow.com/publications/s2012-shading-course/burley/s2012_pbs_disney_brdf_slides_v2.pdf)
* [Ashkimin Shirley BRDF](https://blog.selfshadow.com/publications/s2012-shading-course/burley/s2012_pbs_disney_brdf_slides_v2.pdf)

在 Babylon.js 中，PBR 是通过 PBRMaterial 完成的。该材质包含现代基于物理的渲染所需的所有功能。在此页面上，我们将介绍两个预设的简化版本，可让您快速开始使用 PBR。

您可以从 Babylon.js 主网站找到使用 [PBRMaterial](https://www.babylonjs.com/demos/pbrglossy/) 效果的演示。

![](https://doc.babylonjs.com/img/pbr.jpg)

这两种附加材质是 PBRMetallicRoughnessMaterial 和 PBRSpecularGlossinessMaterial。它们都实现了基于 GLTF 规范的特定约定：

* [金属粗糙度约定](https://github.com/KhronosGroup/glTF/blob/master/specification/2.0/README.md#metallic-roughness-material)
* [镜面光泽度约定](https://github.com/KhronosGroup/glTF/tree/main/extensions/2.0/Archived/KHR_materials_pbrSpecularGlossiness) (译者注：该标准目前已废弃，推荐使用新的 [KHR_materials_specular](https://github.com/KhronosGroup/glTF/blob/main/extensions/2.0/Khronos/KHR_materials_specular/README.md))

## PBRMetallicRoughnessMaterial

该材质基于五个主要属性：

* baseColor / baseTexture：根据金属度的值，基色有两种不同的解释。当材质为金属时，基色为垂直入射 (F0) 下的具体测量反射率值。对于非金属，基色代表材质的反射漫反射颜色。
* metallic：指定材质的金属标量值。也可用于缩放金属纹理的金属度值。
* roughness：指定材质的粗糙度标量值。也可用于缩放金属纹理的粗糙度值。
* MetallicRoughnessTexture：同时包含 B 通道中的金属值和 G 通道中的粗糙度值的纹理，以保持更好的精度。环境光遮挡也可以保存在 R 通道中。
* environmentTexture：纹理

由于您已经非常熟悉Babylon的StandardMaterial，现在我们将仅尝试在这里解决主要差异并作为最简单的设置；您唯一的更改是实例化 PBRMetallicRoughnessMaterial 而不是 StandardMaterial。

````javascript
const pbr = new BABYLON.PBRMetallicRoughnessMaterial("pbr", scene);
````

将此材料应用于您选择的对象，例如：

````javascript
sphere.material = pbr;
````

现在，您可以定义材料的基于物理的值，以获得良好的外观和感觉：

````javascript
pbr.baseColor = new BABYLON.Color3(1.0, 0.766, 0.336);
pbr.metallic = 0;
pbr.roughness = 1.0;
````

[PBR粗糙度](https://playground.babylonjs.com/#2FDQT5)

通过此特定配置，您可以看到根本没有反射（金属设置为 0），也没有镜面反射（粗糙度设置为 1）。

如果我们想引入更多反射，我们可以做相反的事情：

````javascript
pbr.baseColor = new BABYLON.Color3(1.0, 0.766, 0.336);
pbr.metallic = 1.0;
pbr.roughness = 0;
````

但在这种情况下，我们需要一些东西来反思。要定义环境反射，只需添加以下行：

````javascript
pbr.environmentTexture = BABYLON.CubeTexture.CreateFromPrefilteredData("/textures/environment.dds", scene);
````

此调用将创建材质用于产生最终输出的所有必需数据。

[PBR金属反射表面](https://playground.babylonjs.com/#2FDQT5#11)

也许现在有点太反光了，所以让我们添加更多粗糙度以使其看起来更金色：

````javascript
pbr.baseColor = new BABYLON.Color3(1.0, 0.766, 0.336);
pbr.metallic = 1.0;
pbr.roughness = 0.4;
pbr.environmentTexture = BABYLON.CubeTexture.CreateFromPrefilteredData("/textures/environment.dds", scene);
````

[PBR带有粗糙度的金属表面](https://playground.babylonjs.com/#2FDQT5#12)

为了更精确地了解对象的金属感和粗糙度，您还可以指定 MetallicRoughnessTexture：

````javascript
pbr.baseColor = new BABYLON.Color3(1.0, 0.766, 0.336);
pbr.metallic = 1.0; // set to 1 to only use it from the metallicRoughnessTexture
pbr.roughness = 1.0; // set to 1 to only use it from the metallicRoughnessTexture
pbr.environmentTexture = BABYLON.CubeTexture.CreateFromPrefilteredData("/textures/environment.dds", scene);
pbr.metallicRoughnessTexture = new BABYLON.Texture("/textures/mr.jpg", scene);
````

[PBR带有粗糙度金属度贴图的金属表面](https://playground.babylonjs.com/#2FDQT5#13)

## PBRSpecularGlossinessMaterial

该材质基于五个主要属性：

* diffuseColor/diffuseTexture：指定材质的漫反射颜色。
* specularColor：指定材质的镜面颜色。这表明材料的反射程度（无镜像）。
* glossiness：指定材质的光泽度。这表明“反射有多锐利”。
* specularGlossinessTexture：指定材质每像素的镜面反射颜色 RGB 和光泽度 A。
* environmentTexture：纹理

该材质的设置与 PBRMetallicRoughnessMaterial 中使用的设置相当：

````javascript
const pbr = new BABYLON.PBRSpecularGlossinessMaterial("pbr", scene);
pbr.diffuseColor = new BABYLON.Color3(1.0, 0.766, 0.336);
pbr.specularColor = new BABYLON.Color3(1.0, 0.766, 0.336);
pbr.glossiness = 0.4;
pbr.environmentTexture = BABYLON.CubeTexture.CreateFromPrefilteredData("/textures/environment.dds", scene);
````

[PBR光泽度表面](https://playground.babylonjs.com/#Z1VL3V#5)

然后可以使用specularGlossinessTexture（类似MetallicRoughnessTexture纹理）来提供对镜面反射和光泽度的更多控制：

````javascript
pbr.diffuseColor = new BABYLON.Color3(1.0, 0.766, 0.336);
pbr.specularColor = new BABYLON.Color3(1.0, 1.0, 1.0);
pbr.glossiness = 1.0;
pbr.environmentTexture = BABYLON.CubeTexture.CreateFromPrefilteredData("/textures/environment.dds", scene);
pbr.specularGlossinessTexture = new BABYLON.Texture("/textures/sg.png", scene);
````

[PBR带有光泽度贴图的表面](https://playground.babylonjs.com/#Z1VL3V#4)

## 灯光设置

动态灯光是 PBR 设置的重要组成部分。您可以决定不使用灯光，仅使用环境纹理来照亮场景，也可以决定添加其他光源来增强渲染效果。

默认情况下，光强度是使用距光源的距离平方反比计算的。这是一种与现实生活中光线非常接近的衰减类型。因此，距离越远，到达表面所需的强度就越大。

更进一步，您在灯光上定义的强度遵循物理概念：

* 点光源和聚光灯以发光强度定义（candela、m/sr）
* 定向光和半球光的照度（nit、cd/m2）
* 您将找到有关动态光照如何在掌握 PBR 中发挥作用的更多信息，但首先我们看看如何将高动态范围 (HDR) 与 PBR 结合使用