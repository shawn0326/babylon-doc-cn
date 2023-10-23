# 材质简介

材质允许您用颜色和纹理覆盖网格。材质的显示方式取决于场景中使用的灯光以及它的反应方式。

## 对光的反应

材质对光的反应有四种可能的方式。

1. 漫反射 - 在灯光下观察到的材质的基本颜色或纹理；
2. 镜面反射 - 光线赋予材质的高光；
3. 自发光 - 颜色或纹理就像自己发出的光一样；
4. 环境 - 环境背景照明照亮的材质的颜色或纹理。

漫反射和镜面材质需要创建光源。
环境色需要设置场景的环境色，给出环境背景照明。

````javascript
scene.ambientColor = new BABYLON.Color3(1, 1, 1);
````

## 颜色

创建材质

````javascript
const myMaterial = new BABYLON.StandardMaterial("myMaterial", scene);
````

使用diffuseColor、specularColor、emissiveColor 和 ambientColor 中的一种、部分或全部设置材质颜色。请记住，仅当场景环境颜色已设置时，ambientColor 才会应用。

````javascript
const myMaterial = new BABYLON.StandardMaterial("myMaterial", scene);

myMaterial.diffuseColor = new BABYLON.Color3(1, 0, 1);
myMaterial.specularColor = new BABYLON.Color3(0.5, 0.6, 0.87);
myMaterial.emissiveColor = new BABYLON.Color3(1, 1, 1);
myMaterial.ambientColor = new BABYLON.Color3(0.23, 0.98, 0.53);

mesh.material = myMaterial;
````

## 漫反射颜色示例

为了了解材质漫射颜色如何对漫射光颜色做出反应，以下Playground示例展示了不同颜色的材质如何对白色、红色、绿色和蓝色漫射聚光灯做出反应。

[不同颜色的材质对灯光颜色的反应](https://playground.babylonjs.com/#20OAV9#325)


这个反应包括：

* 黄色材质
* 紫色材质
* 青色材质
* 白色材质

下图中还可以看到白色、红色、绿色和蓝色漫射聚光灯。

![](https://doc.babylonjs.com/img/how_to/Materials/spots1.png)

## 环境颜色示例

在下图中，所有球体都由相同的半球光照亮，具有漫反射红色和地面颜色绿色。第一个球体没有环境颜色，中间的球体在其材质上定义了红色环境颜色，右侧的球体具有绿色环境颜色的材质。必须存在的场景环境颜色是白色。

当场景环境颜色分量设置为 0（例如红色）时，无论材质环境颜色中的红色值是多少，都不会产生任何效果。

![](https://doc.babylonjs.com/img/how_to/Materials/ambient1.png)

[使用环境颜色](https://playground.babylonjs.com/#20OAV9#14)

## 透明颜色示例

透明度是通过将材质 alpha 属性设置为 0（不可见）到 1（不透明）来实现的。

````javascript
myMaterial.alpha = 0.5;
````

[透明材质](https://playground.babylonjs.com/#20OAV9#16)

## 纹理

纹理是使用保存的图像形成的。

创建材质

````javascript
const myMaterial = new BABYLON.StandardMaterial("myMaterial", scene);
````

使用diffuseTexture、specularTexture、emissiveTexture 和 ambientTexture 之一、部分或全部设置材质纹理。请注意，应用 ambientTexture 时未设置场景环境颜色。

````javascript
const myMaterial = new BABYLON.StandardMaterial("myMaterial", scene);

myMaterial.diffuseTexture = new BABYLON.Texture("PATH TO IMAGE", scene);
myMaterial.specularTexture = new BABYLON.Texture("PATH TO IMAGE", scene);
myMaterial.emissiveTexture = new BABYLON.Texture("PATH TO IMAGE", scene);
myMaterial.ambientTexture = new BABYLON.Texture("PATH TO IMAGE", scene);

mesh.material = myMaterial;
````

注意：当未指定法线时，Babylon 的标准材质将自动计算法线。

## 纹理示例

在此图像中，所有球体都由相同的半球光照亮，具有漫反射红色和地面颜色绿色。第一个球体具有漫反射纹理，中间的球体具有发光纹理，右侧的球体具有红色漫反射颜色的材质和环境纹理。

[设置了漫反射，发光颜色的纹理和环境纹理](https://playground.babylonjs.com/#20OAV9#15)

## 透明纹理示例

对于颜色，透明度是通过将材质 alpha 属性设置为 0（不可见）到 1（不透明）来实现的。

````javascript
myMaterial.alpha = 0.5;
````

[纹理的透明](https://playground.babylonjs.com/#20OAV9#17)

此外，用于纹理的图像可能已经具有透明度设置，例如来自维基共享资源的这张狗的图片，它具有透明的背景；

![](https://doc.babylonjs.com/img/how_to/Materials/dog.png)

在这个例子中，我们需要将纹理的 hasAlpha 属性设置为 true。

````javascript
myMaterial.diffuseTexture.hasAlpha = true;
````

[背景透明的纹理](https://playground.babylonjs.com/#20OAV9#18)

为了通过正面的透明区域看到立方体的背面，我们必须处理背面剔除。

## 纹理打包器

一些复杂的场景仅一种材质就需要大量纹理。在这种情况下，打包纹理会很方便。必须权衡使用纹理打包程序的优势与固定大小缩放等限制。

[有关创建纹理包的更多信息](https://doc.babylonjs.com/features/featuresDeepDive/materials/advanced/texturePackage)

## 背面剔除

这是一种高效绘制3D模型2D渲染的方法。通常不需要绘制立方体或其他物体的背面，因为它会被正面隐藏。正如您所料，Babylon.js 中的默认设置为 true。在大多数情况下，这有助于保持尽可能高的性能。

看下面的图片，当材质属性 backFaceCulling 为 true 时，你可以看到狗周围的透明区域仍然是透明的，你可以透过它们看到背景。但是，您无法看到背面的图像，因为它们已被剔除（或删除）。当 backFaceCulling 为 false 时，渲染期间不会删除背面，因此可以通过正面的透明区域看到它们。

|Back Face Culling True|Back Face Culling False|
|--|--|
|![](https://doc.babylonjs.com/img/how_to/Materials/bfc2.png)|![](https://doc.babylonjs.com/img/how_to/Materials/bfc1.png)|

[背面剔除的示例](https://playground.babylonjs.com/#YDO1F#20)

## 线框

您可以使用以下命令在线框模式下查看网格：

````javascript
materialSphere1.wireframe = true;
````

![](https://doc.babylonjs.com/img/how_to/Materials/04-3.png)

## 本地文件访问

要记住的重要一点是，出于安全原因，网络浏览器不允许网页访问本地文件。这包括您正在使用的任何纹理文件。您可以使用本地服务器或启用 CORS 的图像托管服务。