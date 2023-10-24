# PBR进阶

## 介绍

PBR 材质的两个简化版本，例如PBRSpecularGlossinessMaterial 和 PBRMetallicRoughnessMaterial 是 PBR 的一个很好的介绍，但是使用 PBRMaterial 本身将允许您使用以下功能更好地控制材质：

* Refraction
* Standard Light Falloff
* LightMaps
* Dedicated image processing

[示例场景 - PBR材质](https://www.babylonjs.com/demos/pbrrough/)

本页讨论了 PBRMaterial 及其简单版本之间的差异。

## 从金属粗糙度到 PBR 材质

为了在金属/粗糙度模式下设置 PBRMaterial，至少必须设置以下属性之一（否则默认情况下在镜面/光泽度下工作）：

* metallic
* roughness
* metallicTexture

要从 PBRMetallicRoughnessMaterial 切换到更大的 PBRMaterial，需要重命名一些属性（在更丰富的材质中尚未完成重命名，以便保持与先前版本的向后兼容性）：

|PBRMetallicRoughnessMaterial|PBRMaterial|
|--|--|
|baseColor|albedoColor|
|baseTexture|albedoTexture|
|metallicRoughnessTexture|metallicTexture|
|environmentTexture|reflectionTexture|
|normalTexture|bumpTexture|
|occlusionTexture|ambientTexture|
|occlusionTextureStrength|ambientTextureStrength|

由于用于金属或粗糙度的通道可以自定义，为了兼容简单材质，您需要添加以下标志：

````javascript
pbr.useRoughnessFromMetallicTextureAlpha = false;
pbr.useRoughnessFromMetallicTextureGreen = true;
pbr.useMetallnessFromMetallicTextureBlue = true;
````

[PBR自定义金属表面](https://playground.babylonjs.com/#2FDQT5#14)

转换完成后，让我们看看此版本上可用的自定义选项：

* useRoughnessFromMetallicTextureAlpha：金属纹理在其 Alpha 通道中包含粗糙度信息。
* useRoughnessFromMetallicTextureGreen：金属纹理在其绿色通道中包含粗糙度信息（useRoughnessFromMetallicTextureAlpha 需要为 false）。
* useMetallnessFromMetallicTextureBlue：金属纹理在其蓝色通道中包含金属信息（默认情况下在红色通道中被视为）。
* useAmbientOcclusionFromMetallicTextureRed：金属纹理在其红色通道中包含环境光遮挡信息。
* useAmbientInGrayScale：环境光遮挡被迫仅从环境纹理的红色通道或金属纹理的红色通道读取。

## 从镜面光泽度到 PBR 材质

在镜面反射/光泽度模式下设置 PBRMaterial 是不同的。
以下属性必须为 null 或未定义：

* metallic
* roughness
* metallicTexture

要从 PBRSpecularGlossinessMaterial 切换到更丰富的 PBRMaterial，还需要重命名一些属性：

|PBRSpecularGlossinessMaterial|PBRMaterial|
|--|--|
|diffuseColor|albedoColor|
|diffuseTexture|albedoTexture|
|specularGlossinessTexture|reflectivityTexture|
|specularColor|reflectivityColor|
|glossiness|microSurface|
|normalTexture|bumpTexture|
|occlusionTexture|ambientTexture|
|occlusionTextureStrength|ambientTextureStrength|

此外，由于用于光泽度的通道可以自定义，为了兼容简单材质，您需要添加以下标志：

````javascript
pbr.useMicroSurfaceFromReflectivityMapAlpha = false;
````

转换完成后，让我们看看此版本上可用的自定义选项：

* microSurfaceTexture：能够将光泽度存储在单独纹理的红色通道上。
* useAlphaFromAlbedoTexture：不透明度信息包含在反照率纹理的 Alpha 通道中。
* useMicroSurfaceFromReflectivityMapAlpha：反射率纹理在其 Alpha 通道中包含微表面或光泽度信息。
* useAmbientInGrayScale：环境光遮挡被迫仅从环境纹理的红色通道或金属纹理的红色通道读取。

## 控制金属粗糙度材质上的镜面反射

镜面反射很大程度上受到菲涅尔效应的影响，菲涅尔效应指出，从表面反射的光量取决于观察角度。表面垂直于视角的点被认为是零菲涅耳或 0 度菲涅耳，可以缩写为 F0。通常，光滑的介电材质会反射 F0 照射到表面的光的 2% 到 5%。大部分与视角平行的表面被视为掠射角，称为 90 度菲涅耳角或 F90。在 F90 时，光滑的介电材质将反射几乎 100% 照射到表面的光线。通常，当使用 PBR 金属粗糙度材质时，F0 的反射率值在着色器中硬编码，介电材质的反射率颜色来自浅色。

然而，有时需要控制 PBR 金属粗糙度材质的镜面反射的功率或颜色。此外，如果没有这种类型的控制，有些物质就无法准确呈现。PBR 镜面光泽度材质可用于这些类型的物质，但有多种方法可以使用金属粗糙度材质来渲染需要修改其镜面反射的物质。

对于 glTF 模型，有一个名为 Khronos 的扩展 [KHR_materials_specular](https://github.com/KhronosGroup/glTF/tree/main/extensions/2.0/Khronos/KHR_materials_specular)，它可以向金属粗糙度材质添加参数specular。此外，该扩展允许使用纹理通过和specularColor来控制这些值。这些参数影响电介质 BRDF，从而改变 F0 反射的镜面功率或颜色，而以前只能使用镜面光泽度材质才能实现。如果glTF 文件使用此扩展，Babylon.js 加载器会将 glTF 材质描述转换为参数设置正确的PBRMaterial。

Babylon.js 构造为金属粗糙度的材质还可以通过PBRMaterial上的多个参数来控制镜面高光。除了材质的基本设置之外，还需要定义这些参数来塑造镜面反射。

* metallicF0Factor：这与KHR_materials_specular中的specularFactor相同。乘以该参数metallicReflectanceColor可控制镜面反射的强度。
* metallicReflectanceColor：这与KHR_materials_specular中的specularColorFactor相同。该参数设置 F0 处的镜面反射颜色。当表面法线向 F90 移动时，该值将插值为白色。
* metallicReflectanceTexture：这是一个纹理，保存metallicReflectanceColorRGB 通道中的值和metallicF0FactorAlpha 通道中的值。在计算最终镜面反射贡献时，纹理分量将与metallicReflectanceColor和相乘。如果在引擎中创作权利metallicF0Factor，则该纹理可以用作两者的单个输入。但是，如果使用使用扩展名创作的 glTF 文件中的纹理，则此参数将按照扩展名中的定义进行操作。这是因为扩展将和定义为两个不同的纹理，并保持 Alpha 通道中镜面反射的强度。如果使用纹理，请将定义的纹理分配给参数并将参数设置为 true。
* reflectanceTexture：这与KHR_materials_specular中的specularColorTexture属性相同。仅当没有分配纹理或设置为 true时，存储在纹理 RGB 通道中的颜色才会相乘metallicReflectanceColor并缩放metallicF0Factor以产生最终的镜面反射贡献。否则，分配给的任何纹理都将使用其 RGB 通道来代替该纹理。metallicReflectanceTexture useOnlyMetallicFromMetallicReflectanceTexturemetallicReflectanceTexture
* useOnlyMetallicFromMetallicReflectanceTexture：如果该参数设置为 true，则在光照计算中仅使用Alpha通道。如果使用为KHR_material_specular扩展创作的纹理，此参数很重要，因为如果 和metallicReflectanceTexture都reflectanceTexture分配了纹理，metallicReflectanceTexture则默认情况下将使用 ，除非useOnlyMetallicFromMetallicReflectanceTexture设置为 true。如果该参数设置为 true，则分配给 的纹理的 RGB 通道reflectanceTexture将用于确定 的值metallicReflectanceColor。

为了帮助说明这些参数如何协同工作，请考虑以下代码片段，它们都是控制金属粗糙度材质中镜面反射的有效方法。

````javascript  
// creating material from scratch using only metallicReflectanceTexture 
// metReflectTex.rgb * metallicReflectanceColor scaled 
// by metReflectTex.a * metallicF0Factor
let pbrMat = new BABYLON.PBRMaterial("pbrMat", scene);
pbrMat.metallicF0Factor = 0.9;
pbrMat.metallicReflectanceColor = new BABYLON.Color3(0.63, 0.12, 0.12);
pbrMat.metallicReflectanceTexture = new BABYLON.Texture("metReflectTex.png", scene);

// creating material from scratch using only reflectanceTexture
// reflectTex.rgb * metallicReflectanceColor scaled by metallicF0Factor
let pbrMat = new BABYLON.PBRMaterial("pbrMat", scene);
pbrMat.metallicF0Factor = 0.9;
pbrMat.metallicReflectanceColor = new BABYLON.Color3(0.63, 0.12, 0.12);
pbrMat.reflectanceTexture = new BABYLON.Texture("reflectTex.png", scene);

// creating material from KHR_material_specular textures
// specularColorTexture.rgb * metallicReflectanceColor scaled 
// by specularTexture.a * metallicF0Factor
let pbrMat = new BABYLON.PBRMaterial("pbrMat", scene);
pbrMat.metallicF0Factor = 0.9;
pbrMat.metallicReflectanceColor = new BABYLON.Color3(0.63, 0.12, 0.12);
pbrMat.metallicReflectanceTexture = new BABYLON.Texture("specularTexture.png", scene);
pbrMat.reflectanceTexture = new BABYLON.Texture("specularColorTexture.png", scene);
pbrMat.useOnlyMetallicFromMetallicReflectanceTexture = true;
````

[使用 PBRMaterial 的镜面反射](https://playground.babylonjs.com/#FU2ZQ7)

## 不透明度

反射的另一个有趣的补充是能够将反射最明亮的部分保留在透明表面上...是的，这没有多大意义...实际上，如果您在晚上从明亮的房间透过窗户看，您可以在玻璃上看到灯光或电视的反射。这与 PBR 材质中的反射相同。添加了一个特殊属性pbr.useRadianceOverAlpha = true;来允许您控制此效果。在透明度之上不仅可以看到反射（又名辐射），还可以看到镜面高光。

[PBR 中的不透明度](https://playground.babylonjs.com/#19JGPR#13)

````javascript
glass.reflectionTexture = hdrTexture;
glass.alpha = 0.5;
````

可以通过以下属性关闭此行为：

````javascript
useRadianceOverAlpha = false;
useSpecularOverAlpha = false;
````

## 折射（向后兼容）

折射有点像反射（请纯粹主义者，现在不要杀我，我刚只说了一点），因为它严重依赖环境来改变材质的外观。基本上，如果反射可以比作在湖面上看到的太阳和云彩，那么折射就是在水面下（通过水）看到形状怪异的鱼。下一节有更多关于折射的内容。

由于折射相当于您如何看穿不同材质的边界，因此可以通过 BJS 中的透明度来控制效果。一个特殊的属性可以帮助您做到这一点，只需设置pbr.linkRefractionWithTransparency=true;然后 alpha 将控制材质的折射程度。将其设置为 false 会使 alpha 控制默认透明度。

[PBR 中的折射](https://playground.babylonjs.com/#19JGPR#12)

````javascript
const glass = new BABYLON.PBRMaterial("glass", scene);
glass.reflectionTexture = hdrTexture;
glass.refractionTexture = hdrTexture;
glass.linkRefractionWithTransparency = true;
glass.indexOfRefraction = 0.52;
glass.alpha = 0; // Fully refractive material
````

由于能量守恒，您仍然可以注意到材质上的一些反射。请注意，从 4.0 开始，您应该依赖下一节的设置来定义影响材质表面下发生的情况的所有事物。但不用担心，我们将保留当前设置以实现向后兼容性。

## 次表面

材质的次表面部分定义了表面以下发生的一切。目前它支持折射和半透明。

## 折射

我不会在这里重新定义折射分量，因为它已在上一节中讨论过，但仅强调主要差异。

启用折射可以通过子表面部分上的标志来完成：

[在 PBR 中启用折射](https://playground.babylonjs.com/#FEEK7G#17)

````javascript
const pbr = new BABYLON.PBRMaterial("pbr", scene);
sphere.material = pbr;

pbr.metallic = 0;
pbr.roughness = 0;

pbr.subSurface.isRefractionEnabled = true;
pbr.subSurface.refractionIntensity = 0.8;
````

和以前一样，您可以控制折射率：

[控制折射率](https://playground.babylonjs.com/#FEEK7G#24)

````javascript
const pbr = new BABYLON.PBRMaterial("pbr", scene);
sphere.material = pbr;

pbr.metallic = 0;
pbr.roughness = 0;

pbr.subSurface.isRefractionEnabled = true;
pbr.subSurface.indexOfRefraction = 1.5;
````

请注意，这里的折射率代表您可以在术语中找到的值，而不是像传统设置中那样的倒数。

您可以通过配置两个属性来控制材质的色调（表示其表面以下的颜色）：

tintColor：定义色调的颜色。
tintColorAtDistance：定义表面以下的距离应为定义的颜色（通过啤酒朗伯定律模拟吸收）

[PBR中的颜色控制](https://playground.babylonjs.com/#FEEK7G#25)

````javascript
const pbr = new BABYLON.PBRMaterial("pbr", scene);
sphere.material = pbr;

pbr.metallic = 0;
pbr.roughness = 0;

pbr.subSurface.isRefractionEnabled = true;
pbr.subSurface.indexOfRefraction = 1.5;
pbr.subSurface.tintColor = BABYLON.Color3.Teal();
````

默认情况下，材质的厚度被理解为 maxThicknesssubSurface 的值。您可以通过厚度图轻松更改厚度：

````javascript
const pbr = new BABYLON.PBRMaterial("pbr", scene);
sphere.material = pbr;

pbr.metallic = 0;
pbr.roughness = 0;

pbr.subSurface.isRefractionEnabled = true;
pbr.subSurface.indexOfRefraction = 1.5;
pbr.subSurface.tintColor = BABYLON.Color3.Teal();

pbr.subSurface.thicknessTexture = texture;
pbr.subSurface.minimumThickness = 1;
pbr.subSurface.maximumThickness = 10;
````

每个像素的实际厚度将为 = 最小厚度 + 厚度纹理.r * 最大厚度。这有助于将实际值限制在纹理定义的最小值和最大值之间。

## 半透明性

TODO