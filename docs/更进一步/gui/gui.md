# Babylon GUI

## 如何使用BabylonGUI

Babylon.js GUI 库是一个可用于生成交互式用户界面的扩展。它构建在 DynamicTexture 之上。

最新版本可以在我们的 CDN 上找到：https://cdn.babylonjs.com/gui/babylon.gui.js。

源代码可在 Babylon.js 主存储库中找到：https://github.com/BabylonJS/Babylon.js/tree/master/packages/dev/gui。

您可以在这里找到完整的演示：https://www.babylonjs.com/demos/gui/

请注意，除了下面描述的 Babylon 2D GUI 系统之外，通过 Babylon.js v3.3 及更高版本，您还可以使用 3D GUI 系统。这两个系统都可以用于满足您项目的不同需求。

## 介绍

Babylon.GUI 使用 DynamicTexture 生成功能齐全的用户界面，该界面灵活且 GPU 加速。

## 高级动态纹理

要开始使用 Babylon.GUI，您首先需要一个 AdvancedDynamicTexture 对象。

Babylon.GUI 有两种模式：

### 全屏模式

在此模式下，Babylon.GUI 将覆盖整个屏幕，并将重新缩放以始终适应您的渲染分辨率。它还会拦截点击（包括触摸）。要在全屏模式下创建 AdvancedDynamicTexture，只需运行以下代码：

````javascript
const advancedTexture = BABYLON.GUI.AdvancedDynamicTexture.CreateFullscreenUI("myUI");
````

默认情况下，渲染分辨率和纹理大小之间的比率为 1。但是您可以使用 advanceTexture.renderScale 强制其为不同的值。例如，如果您想要更清晰的文本，这可能很有用。

前景和背景：全屏模式可以在场景的前景或背景中渲染。可以这样设置：

````javascript
// true == foreground (default)
// false == background
const advancedTexture = BABYLON.GUI.AdvancedDynamicTexture.CreateFullscreenUI("myUI", foreground? : Boolean );
// it can also be changed on the go:
advancedTexture.isForeground = false;
````

**请注意，每个场景仅允许一个全屏模式 GUI**

全屏模式不适用于 WebVR，因为它是纯 2D 渲染。对于 WebVR 场景，您必须使用下面的纹理模式。

### 纹理模式

在此模式下，BABYLON.GUI 将用作给定网格的纹理。您必须定义纹理的分辨率。要在纹理模式下创建 AdvancedDynamicTexture，只需运行以下代码：

````javascript
const advancedTexture2 = BABYLON.GUI.AdvancedDynamicTexture.CreateForMesh(myPlane, 1024, 1024);
````

下面是一个简单纹理模式 GUI 的示例：

[纹理模式GUI示例](https://playground.babylonjs.com/#ZI9AK7#1)

这是相同的示例，但现在使用“billboardMode = all”，其中 GUI 将始终面向相机：[面向相机的纹理模式示例](https://playground.babylonjs.com/#ZI9AK7#1214)

请注意，在复杂的网格上处理指针移动事件的成本可能很高，因此您可以使用第四个参数关闭支持指针移动事件：

````javascript
const advancedTexture2 = BABYLON.GUI.AdvancedDynamicTexture.CreateForMesh(myPlane, 1024, 1024, false);
````

一旦有了 AdvancedDynamicTexture 对象，您就可以开始添加控件。

## 从Snippet服务器加载

以下是从Snippet服务器加载 AdvancedDynamicTexture 的示例：

[从Snippet服务器加载GUI的示例](https://playground.babylonjs.com/#AJA7KA#50)

## 调试

从 Babylon.js v4.0 开始，新的Inspector可以通过显示边界信息并让您动态更改属性来帮助调试 GUI：[Inspector文档]

## 常用特性

### 事件

**请注意，控件需要具有 control.isPointerBlocker = true 才能正确处理所有指针事件。默认情况下，此属性在明显的控件（例如按钮）上设置，但如果您想在图像等控件上使用它，则必须将其打开**。

所有控件都有以下可监听器：

|监听器|注释|
|--|--|
|onPointerMoveObservable|当光标移动到控件上时引发。仅在全屏模式下可用|
|onPointerEnterObservable|当光标进入控件时引发。仅在全屏模式下可用|
|onPointerOutObservable|当光标离开控件时引发。仅在全屏模式下可用|
|onPointerDownObservable|当指针位于控件上时引发。|
|onPointerUpObservable|当指针位于控件上时引发。|
|onPointerClickObservable|单击控件时引发。|
|onClipboardObservable|触发剪贴板事件时引发。|

要使用剪贴板事件，首先需要通过调用 AdvancedDynamicTexture 实例上的 registerClipboardEvents 来启用它们，该实例会将剪切、复制、粘贴事件注册到画布上。启用后，可以通过 ctrl/cmd + c 进行复制、ctrl/cmd + v 进行粘贴、ctrl/cmd + x 进行剪切来触发它们，并且它们将始终监听画布。如果您有任何其他具有相同键绑定的操作，则可以通过调用 unRegisterClipboardEvents 来防止默认触发这些事件，这将从画布上取消注册它们。

以下是如何使用剪贴板监听器的示例：

* [创建新Mesh](https://playground.babylonjs.com/#S0IW99#1)
* [要从剪贴板数据创建新文本块](https://playground.babylonjs.com/#AY28VL#4)

您还可以定义控件对事件不可见（例如，您可以单击它）。为此，只需调用 control.isHitTestVisible。

请注意，onPointerMoveObservable、onPointerDownObservable、onPointerUpObservable、onPointerClickObservable 将接收包含指针坐标的 Vector2 参数。如果你想获取本地控件空间中的指针坐标，你必须调用control.getLocalCooperatives(coordinates)

以下是如何使用监听器的示例：[监听器示例](https://playground.babylonjs.com/#XCPP9Y#121)

以下是如何使用 onPointerClickObservable 的示例：[onPointerClickObservable示例](https://playground.babylonjs.com/#7RH606)

## 对齐

您可以使用以下属性定义控件使用的对齐方式：

|属性|默认值|注释|
|--|--|--|
|horizontalAlignment|2|可以设置为 left, right or center.|
|verticalAlignment|2|可以设置为 top, bottom and center.|

枚举值可以取自 BABYLON.GUI.Control.HORIZONTAL_ALIGNMENT_* 和 BABYLON.GUI.Control.VERTICAL_ALIGNMENT_*。

以下是如何使用对齐的示例：[对齐示例](https://playground.babylonjs.com/#XCPP9Y#13)

// TODO
