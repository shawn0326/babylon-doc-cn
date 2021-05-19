入门 - 第二章 - Mesh的放置
===

## 放置并缩放一个Mesh

## 尺寸

一些mesh，例如Box，可以在创建的时候设置一些属性。

````javascript
const box = BABYLON.MeshBuilder.CreateBox("box", {width: 2, height: 1.5, depth: 3})
````

假如mesh已经创建完成了，并且创建的时候并没有传入尺寸相关的参数，也可以通过缩放来缩放尺寸。

````javascript
const box = BABYLON.MeshBuilder.CreateBox("box", {}); //unit cube
box.scaling.x = 2;
box.scaling.y = 1.5;
box.scaling.z = 3;
````

````javascript
const box = BABYLON.MeshBuilder.CreateBox("box", {}); //unit cube
box.scaling = new BABYLON.Vector3(2, 1.5, 3);
````

从上面的代码可以看出，`scaling`属性是一个vector类型，有x，y，z三个属性。

上面的两段代码创建了相同尺寸的Box。

## 位置

对于大多数的mesh来说，`position`都被放置在坐标系的原点。position属性的类型也是vector类型并拥有x，y，z三个属性，所以下面两段代码均可以将box放在同样的位置。

````javascript
box.position.x = -2;
box.position.y = 4.2;
box.position.z = 0.1;
````

````javascript
box.position = new BABYLON.Vector3(-2, 4.2, 0.1);
````

在下面这个playground中，我们用position属性将三个Box放在不同的位置，每个Box都以不同的方式进行了缩放。因为每个Box的高度都是1.5，所以为了把Box放置在地面上，position.y都被设置成0.75。

[Positioning Meshes](https://playground.babylonjs.com/#KBS9I5#68)

## 旋转

与缩放和位置属性类似，`rotation`属性类型也是一个由x,y,z组成的vector类型。但是，考虑到我们是初次搭建这个世界，我们仅绕一个轴进行旋转，因为围绕三个轴设置旋转可能会不太容易理解。

````javascript
box.rotation.y = Math.PI / 4;
box.rotation.y = BABYLON.Tools.ToRadians(45);
````

[Rotating Meshes](https://playground.babylonjs.com/#KBS9I5#69)

现在，我们可以通过更改缩放，位置和旋转，使Box作为建筑更加多样化。在放入更多的Box到场景之前，先让我们使它更像一个建筑。