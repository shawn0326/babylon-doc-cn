创建参数化网格体
===

## 参数化形状

这一类参数之所以叫做“参数化形状”是因为它们的形状是由一个Vector3的数组来定义的，它们的形状本质上是不规律的。您可以在3D空间中生成一系列连续或非连续的线。您也可以在2D空间中横向生成不规则的凸多边形或凹多边形，然后在3D空间中竖向挤出。也可以沿着3D空间中的路径拉伸2D形状，并沿着路径自定义其旋转和缩放。您也可以定义一个2D形状并旋转挤出一个旋转对称的形状。

通过下列方法之一创建这种mesh

````javascript
const mesh = BABYLON.MeshBuilder.Create<MeshType>(name, options, scene);
const mesh = BABYLON.MeshBuilder.Extrude<MeshType>(name, options, scene);
````

// TODO