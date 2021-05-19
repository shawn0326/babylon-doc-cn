
入门 - 第二章 - 合并Mesh
===

## 通过MergeMeshes方法合并多个mesh

合并两个或多个mesh很简单：

````javascript
const combined = BABYLON.Mesh.MergeMeshes(Array_of_Meshes_to_Combine)
````

在我们当前例子中：

````javascript
const house = BABYLON.Mesh.MergeMeshes([box, roof])
````

[在你的场景中合并Mesh](https://playground.babylonjs.com/#KBS9I5#75)

![](https://doc.babylonjs.com/_next/image?url=%2Fimg%2Fgetstarted%2Fhouse5.png&w=1920&q=75)

首先注意到的是，整个房间的材质都变成其中一种材质了。幸运的是，我们可以通过设置MergeMeshes方法中的multiMultiMaterial参数来修正这个问题，但不幸的是，这个参数在一长串参数的最末尾。我们将代码改为：

````javascript
const house = BABYLON.Mesh.MergeMeshes([box, roof], true, false, null, false, true);
````

在这里比较重要的参数有两个，第二个为true的参数表示合并mesh之后会释放旧的mesh，最后一个为true的参数表示合并后的mesh会使用多材质对应到原有的部分，保持样式不变。

[合并Mesh并保持材质独立](https://playground.babylonjs.com/#KBS9I5#76)

![](https://doc.babylonjs.com/_next/image?url=%2Fimg%2Fgetstarted%2Fhouse3.png&w=1920&q=75)

在考虑如何拷贝房屋模型之前：我们先会学习如何导出模型，以及如何导入Babylon.js或者其它软件生成的模型；还有如何在自己的网站上展示场景或者模型。