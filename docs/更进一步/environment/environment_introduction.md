# 环境

您如何为您的游戏或应用程序实现良好且适当的现实环境？我们将从简单的场景 clearColor（背景色）开始，然后简要介绍场景 ambientColor，然后是 6 纹理天空盒，然后是雾来为您的场景提供深度幻觉。

![](https://doc.babylonjs.com/img/how_to/13.JPG)

*A picture showing Babylon.js fog in action*

## 怎么做

稍后我们将讨论这种漂亮的雾效果。首先，我想向您介绍场景类对象的两个有趣的属性：

* scene.clearColor - 更改“背景”颜色。
* scene.ambientColor - 更改多种效果中使用的颜色，包括环境照明。

它们都非常有用，并且本身就很强大。

### 改变背景颜色（scene.clearColor）

scene对象上的“clearColor”属性是最基本的环境属性/调整。简单地说，这就是您更改场景背景颜色的方式。这是它是如何完成的：

````javascript
scene.clearColor = new BABYLON.Color3(0.5, 0.8, 0.5);
````

或者，也许您想使用我们的一种预设颜色并避免使用 new 关键字：

````javascript
scene.clearColor = BABYLON.Color3.Blue();
````

此颜色和属性不会用于计算网格、材质、纹理或其他任何内容的最终颜色。它只是场景的背景颜色。简单的。

### 改变环境颜色（scene.ambientColor）

相反，场景对象的 ambientColor 属性是一个非常强大且有影响力的环境属性/调整。首先，让我们看一下它的语法：

````javascript
scene.ambientColor = new BABYLON.Color3(0.3, 0.3, 0.3);
````

如您所见，它使用与 clearColor 相同的格式进行设置，但在确定场景项的最终颜色的大量计算中使用了 ambientColor。主要是，它与网格的 StandardMaterial.ambientColor 结合使用以确定网格材质的最终环境颜色。

你会发现当没有scene.ambientColor的时候，那么StandardMaterial.ambientColor和StandardMaterial.ambientTexture也将不会生效。设置某个值的 scene.ambientColor，如上例，则 StandardMaterial.ambientColor/StandardMaterial.ambientTexture 将在您应用的Mesh上生效。

默认情况下，scene.ambientColor 设置为 Color3(0, 0, 0)，这意味着没有 scene.ambientColor。

（有关更多信息，请参阅我们的教程[标准材质大显身手](https://www.eternalcoding.com/babylon-js-unleash-the-standardmaterial-for-your-babylon-js-game/)中关于 ambientColors 的部分。）

### 天空盒

为了营造美丽晴朗天空的完美幻觉，我们将创建一个简单的盒子，但具有特殊的纹理。有两种创建天空盒的方法。让我们从手动的开始，了解引擎盖下的工作原理，然后我们将能够使用自动的。

TODO
