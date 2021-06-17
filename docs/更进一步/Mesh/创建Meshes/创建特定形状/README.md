创建特定形状
===

## 常用3D形状

这些常用的形状涵盖了2D形状，3D形状。2D形状例如横向的Plane（即ground）或者纵向的Plane等。3D形状例如长方体，球形，椭圆体和圆环等。

创建这些mesh通过调用：

````javascript
const mesh = BABYLON.MeshBuilder.Create<MeshType>(name, options, scene);
````

其中`options`参数对象的属性根据mesh的类型而有所不同。使用空对象`{ }`，mesh将默认为给定尺寸，通常为单位尺寸。 `scene`参数是可选的，默认为当前场景。

其中两个比较常用的属性是*updatable*和*instance*。当options的*updatable*属性为true时，可以更改形状定义属性内的值，并通过使用*instance*属性指定的对象实例来接收这些新值，如下画线的示例所示。

````javascript
// 通过vector3的数组myPoints来创建线
const myLines = BABYLON.MeshBuilder.CreateLines("lines", {points: myPoints});
// 更新其中一些或所有点的值
myPoints[1] = new BABYLON.Vector3(1, 2, 3);
// 通过instance属性来指定更新的mesh
myLines = BABYLON.MeshBuilder.CreateLines("lines", {points: myArray, instance: myLines});
````

该方法允许更改数组中的每一项的值，但注意不能更改数组的长度。所有的参数化mesh，除了`Lathe`，`CreatePolygon`和`ExtendPolygon`，都可以通过该方法进行更改。

目前也可以这样使用：

````typescript
const mesh = BABYLON.Mesh.Create<MeshType>(name, required_param1, required_param2, ..., scene, optional_parameter1, ........);
````

当以上面这种方式调用时，`scene`参数是必须的。这种方式已经在逐步废弃，但你仍可以在一些示例中看到这种写法。

## 接下来要学习

1. [立方体](./立方体/README.md)
2. [平铺立方体](./平铺立方体/README.md)
3. [球体](./球体/README.md)
4. [圆柱体](./圆柱体/README.md)
5. [胶囊体](./胶囊体/README.md)
6. [平面](./平面/README.md)
7. [平铺平面](./平铺平面/README.md)
8. [圆盘](./圆盘/README.md)
9. [圆环](./圆环/README.md)
10. [圆环结](./圆环结/README.md)
11. [地面](./地面/README.md)
12. [高度地面](./高度地面/README.md)
13. [更多关于高度贴图](./更多关于高度贴图/README.md)
14. [平铺地面](./平铺地面/README.md)