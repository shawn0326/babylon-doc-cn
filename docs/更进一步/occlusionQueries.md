# 遮挡查询

## 介绍

Babylon.js v3.1 引入了一个新特性：Occlusion Queries。遮挡查询检测网格在当前场景中是否可见，并基于该网格是否绘制。当您在场景中有一个昂贵的对象并且您想要确保它在相机可见并且不在任何不透明对象后面时将被绘制时，遮挡查询很有用。 BabylonJs 使用 AbstractMesh 类中的属性 occlusionType 提供了 Occlusion Queries 的实现。

## 遮挡查询如何工作

Babylon.js 引擎在绘制对象之前在目标网格上绘制一个浅色透明边界框，并创建一个查询以检查边界框是否可见。如果框可见，则绘制对象 如果不可见，则不绘制对象，遮挡查询是异步的，通常对象的查询结果在当前帧中不可用，因此对象是根据查询结果绘制的与前一帧相比，除非您的 FPS 太低，否则用户不会注意到差异。

[遮挡查询示例](https://playground.babylonjs.com/#QDAZ80#5)

## 基础

在Mesh上使用遮挡查询

````javascript
const sphere = BABYLON.MeshBuilder.CreateSphere("sphere1", { segments: 16, diameter: 2 }, scene);
sphere.occlusionType = BABYLON.AbstractMesh.OCCLUSION_TYPE_OPTIMISTIC;
````

有关 occlusionType 和支持的算法的更多信息，请查看 AbstractMesh 类文档。

如果您的对象默认位于不透明对象后面，您可以将属性 isOccluded 设置为 true，这样 Babylon.js 引擎将不会决定是否渲染，直到从 WebGl 引擎检索到查询结果。

````javascript
sphere.isOccluded = true;
````

## 高级

如前所述，遮挡查询结果是异步的，可能需要一些时间才能获得结果，因此对象需要加载许多帧以等待查询结果。在这种情况下，您可以使用属性 occlusionRetryCount 来设置查询中断之前等待的帧数。一旦中断发生，您将需要决定是绘制对象还是保持其状态，因此使用属性 occlusionType，因为您有 2 个选项

1. OCCLUSION_TYPE_OPTIMISTIC: 如果发生中断，此选项将渲染网格。
2. OCCLUSION_TYPE_STRICT: 此选项将恢复对象的最后状态，无论是可见的继续可见还是隐藏的继续隐藏。

作为使用 restrict 和 optimistic 的场景，如果您的场景中有 2 个昂贵的对象，其中一个是必须渲染的对象，因此您可以设置 occlusionRetryCount 并将 occlusionType 设置为 optimistic 以便在查询结果不可用时渲染您的对象。如果您的对象可以等到查询可用，请不要设置 occlusionRetryCount 或在将 occlusionType 用作 Strict 时设置属性，这样如果对象在上一个场景中渲染，则在当前场景中再次渲染它，否则将其隐藏。

在 Babylon.js 中，您还可以使用属性 occlusionQueryAlgorithmType 设置 WebGl 遮挡查询算法类型，以获取更多信息以查看 AbstractMesh 类文档。

您可以在此处找到在线演示：[高级遮挡查询](https://playground.babylonjs.com/#QDAZ80#3)