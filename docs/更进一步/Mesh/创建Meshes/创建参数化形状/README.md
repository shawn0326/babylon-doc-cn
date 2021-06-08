创建参数化网格体
===

## 参数化形状

这一类参数之所以叫做“参数化形状”是因为它们的形状是由一个Vector3的数组来定义的，它们的形状本质上是不规律的。您可以在3D空间中生成一系列连续或非连续的线。您也可以在2D空间中横向生成不规则的凸多边形或凹多边形，然后在3D空间中竖向挤出。也可以沿着3D空间中的路径拉伸2D形状，并沿着路径自定义其旋转和缩放。您也可以定义一个2D形状并旋转挤出一个旋转对称的形状。

通过下列方法之一创建这种mesh

````javascript
const mesh = BABYLON.MeshBuilder.Create<MeshType>(name, options, scene);
const mesh = BABYLON.MeshBuilder.Extrude<MeshType>(name, options, scene);
````

其中option对象的属性是根据网格类型变化的。scene参数是可选的，默认为当前场景。option对象属性中有一个或多个Vector3数据来定义路径或轮廓，这些一般都是必须设置的。

所有这些参数化形状，除了CreateLathe，CreatePolygon和ExtrudePolygon，options中都有一个*instance*属性，如果创建的时候*updatable*设置为true，这个属性可以用来更新形状。但你只能更改数据数组中的内容，而不能更改数组的长度。

在可能的情况下，playgrounds将会演示两个例子，第一个创建mesh，第二个使用instance选项对mesh进行更新。注释掉后者可以查看mesh原始的形状。

仍然可以通过 “*Mesh”方法创建这些形状，形如：

````javascript
const mesh = BABYLON.Mesh.Create<MeshType>(name, required_param1, required_param2, ..., scene, optional_parameter1, ........);
const mesh = BABYLON.Mesh.Extrude<MeshType>(name, required_param1, required_param2, ..., scene, optional_parameter1, ........);
````

当使用可选参数的时候，scene参数是必须的。你仍然会在playgrounds中找到这种代码。