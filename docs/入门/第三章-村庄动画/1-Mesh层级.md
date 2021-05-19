入门 - 第三章 - Mesh层级
===

## 父子层级

我们将添加一个简单的汽车并操纵它穿越我们的村庄。

不管汽车多简单，也必须要有轮子，而且轮子必须连接到车身上。

![](https://doc.babylonjs.com/_next/image?url=%2Fimg%2Fgetstarted%2Fcarmodel.png&w=1920&q=75)

如果我们使用merge的方式合并mesh，那么轮子就不能转动了。所以这里我们把车身设置为车轮的父级。

````javascript
messhChild.parent = meshParent
````

父级的位移，缩放和旋转操作也会被子级所继承。设置子级的位置，将会作用在父级的坐标系下。设置子级的旋转和缩放将会作用在子级的本地坐标系下。

你可以在下面的playground中修改某些值，来测试父子层级的影响。

[理解父子层级](https://playground.babylonjs.com/#GMEI6U)

现在我们已经准备好建造一辆汽车，然后使它动起来。