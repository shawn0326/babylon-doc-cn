创建网格体
===

## 创建不同类型的网格体

您可以创建各种不同种类的网格体。这些种类主要分为四大类，对于前三类Bybylon.js都有对应的创建方法。

特定形状 - 这种类型的网格体通常会有一个约定俗成的名字。例如：box，sphere，cone和plane。这些方法通常的命名格式是Create<MeshType>。

参数化形状 - 这些形状没有常用名，而是通过数据集描述的，并且往往具有不规则的形状。比如包括带状，挤出形状和不规则的凹凸多边形。这些创建方法主要使用Create<MeshType>，或Extrude<MeshType>的格式命名。

多面体 - 3D标准形状，包括八面体，十二面体，二十面体和二十面体。创建这些形状的方法是createPolyhedron，参数需传入用于区分基本形状类型的数字和用于更广泛形状描述的顶点数组；另外还有createIcoSphere方法。

自定义 - 通过构建一系列顶点围成的三角面，您可以定义任何您想要的形状。

当前创建mesh的方法都在MeshBuilder上。

````javascript
const mesh = BABYLON.MeshBuilder.Create<MeshType>(name, options, scene);
````

options参数根据创建mesh类型的不同，可以传入不同的属性，也可以直接传入空对象{}。scene参数是可选的，默认是当前场景。

还有一个旧版本的方法，为了兼容依旧保留了下来，使用Mesh上的方法和一串参数来创建。

````javascript
const mesh = BABYLON.Mesh.Create<MeshType>(name, required_param1, required_param2, ..., scene, optional_parameter1, ........);
````

当使用这个方法的时候，scene参数是必须传入的。在playground的某些示例中你可能仍然会见到这个方法。

---

## 接下来要学习

1. 创建特定形状
2. 创建参数化形状
3. 创建多面体
3. 创建自定义网格体