# 创建导航网格

## 如何使用导航网格

有很多情况下可以使用导航网格：AI 和路径查找、替代物理碰撞检测（只允许玩家在可能的地方行走，而不是使用碰撞检测）以及 Babylon.js 用户会发现的许多其他情况。

首先，创建导航插件：

```javascript
let navigationPlugin = new BABYLON.RecastJSPlugin();
```

为代理约束准备一些参数（如下所述）

```javascript
const parameters = {
  cs: 0.2,
  ch: 0.2,
  walkableSlopeAngle: 35,
  walkableHeight: 1,
  walkableClimb: 1,
  walkableRadius: 1,
  maxEdgeLen: 12,
  maxSimplificationError: 1.3,
  minRegionArea: 8,
  mergeRegionArea: 20,
  maxVertsPerPoly: 6,
  detailSampleDist: 6,
  detailSampleMaxError: 1,
};
```

调用导航网格生成函数，并传入参数和网格列表

```javascript
navigationPlugin.createNavMesh([groundMesh, wallMesh1, wallMesh2, stair1, stair2], parameters);
```

就是这样！现在你可以使用导航网格与人群系统一起工作或进行查询。

你还可以选择显示导航网格，以确保它符合你的空间约束。

```javascript
navmeshdebug = navigationPlugin.createDebugNavMesh(scene);
const matdebug = new BABYLON.StandardMaterial("matdebug", scene);
matdebug.diffuseColor = new BABYLON.Color3(0.1, 0.2, 1);
matdebug.alpha = 0.2;
navmeshdebug.material = matdebug;
```

[简单的导航网格计算](https://playground.babylonjs.com/#KVQP83#0)

## 参数

cs - 网格被体素化以计算可行走的导航网格。此参数以世界单位定义一个体素的宽度和深度。

ch - 与 cs 类似，但用于体素的高度。

walkableSlopeAngle - 最大可行走坡度的角度（以度为单位）。

walkableHeight - 允许行走的高度（以体素单位表示）。

walkableClimb - 可以攀爬的高度差（以体素单位表示）。

walkableRadius - 代理的半径（以体素单位表示）。

maxEdgeLen - 网格边界轮廓边缘的最大允许长度（以体素单位表示）。

maxSimplificationError - 简化轮廓的边缘与原始轮廓的最大偏离距离（以体素单位表示）。

minRegionArea - 允许形成孤立岛屿区域的最小单元数（以体素单位表示）。

mergeRegionArea - 如果可能，任何跨度计数小于此值的区域将与较大的区域合并（以体素单位表示）。

maxVertsPerPoly - 在轮廓到多边形转换过程中生成的多边形允许的最大顶点数。必须大于 3。

detailSampleDist - 设置生成细节网格时使用的采样距离（以世界单位表示）。

detailSampleMaxError - 细节网格表面与高度场数据的最大偏离距离（以世界单位表示）。

你可以在[Recast 文档](https://recastnav.com/structrcConfig.html)中找到更多关于这些参数的信息。

## 查询

基本上，查询函数通过导航网格帮助获取约束点和向量。

```javascript
getClosestPoint(position: Vector3): Vector3;
getRandomPointAround(position: Vector3, maxRadius: number): Vector3;
moveAlong(position: Vector3, destination: Vector3): Vector3;
```

分别为：

* 获取导航网格上接近一个世界位置参数的点。 
* 在导航网格上获取一个位于最大半径圆内的随机世界位置。 
* 通过导航网格约束一个线段，并返回终点的世界位置。类似于在导航网格上行走并在边缘停止。 

当查询找不到有效解决方案时，返回值为 (0,0,0)。

这些函数使用边界框来查询世界。返回的解决方案在该边界内。要正确设置默认框的范围以获得更精细或更广泛的结果，请调用：

```javascript
setDefaultQueryExtent(extent: Vector3): void;
```

如果查询返回的点距离预期结果太远，请使用较小的范围。

可以获取用于导航的路径，该路径以点数组的形式表示。用户可以使用此数组来绘制预测路径、触发事件等。

```javascript
const pathPoints = navigationPlugin.computePath(crowd.getAgentPosition(agent), navigationPlugin.getClosestPoint(destinationPoint));
pathLine = BABYLON.MeshBuilder.CreateDashedLines("ribbon", { points: pathPoints, updatable: true, instance: pathLine }, scene);
```

## 烘焙结果

构建导航网格可能会消耗大量的 CPU 和网络资源。为了降低下载大小和所需的 CPU，可以将导航网格计算结果烘焙成字节流。稍后可以恢复该字节流以重新获得导航网格。

要获取计算后的导航网格的二进制表示：

```javascript
const binaryData = navigationPlugin.getNavmeshData();
```

binaryData 是一个 Uint8Array，你可以将其保存到文件中。例如，要将 Uint8Array 恢复为导航网格：

```javascript
navigationPlugin.buildFromNavmeshData(uint8array);
```

## Web Worker

构建导航网格可能会消耗大量时间和资源。对于许多用例，可能需要将导航网格计算委托给 Web Worker。这样工作将并行进行，使引擎能够无延迟地显示内容。

要启用 Web Worker，请指定要使用的 Web Worker.js 脚本 URL：

```javascript
let navigationPlugin = new BABYLON.RecastJSPlugin();
navigationPlugin.setWorkerURL("workers/navMeshWorker.js");
```

默认的 Web Worker 提供在此 URL：https://github.com/BabylonJS/Babylon.js/blob/master/packages/tools/playground/public/workers/navMeshWorker.js

然后，为 createNavMesh 方法提供一个完成回调。当导航网格计算完成并准备好由插件使用时，将调用此回调。

```javascript
navigationPlugin.createNavMesh([staticMesh], navmeshParameters,(navmeshData) =>
{
    console.log("got worker data", navmeshData);
    navigationPlugin.buildFromNavmeshData(navmeshData);
    ...
```

navMeshData 是导航网格的二进制版本，已准备好进行序列化，可以保存或流式传输。用户需要调用 buildFromNavmeshData 来反序列化数据。一旦导航网格完全加载，就可以创建人群、查询导航网格等。

性能注意事项：导航网格是从几何数据构建的。如果需要多个网格，它们的几何体将在将几何位置和索引传递给 Recast 之前合并。这部分代码可能会消耗大量 CPU，且由于依赖关系、复制和内存占用，无法在 Worker 中完成。

使用 Web Worker 的示例：[Web Worker 示例](https://playground.babylonjs.com/#TN7KNN#2)

## NPM

加载 Recast-Detour NPM 模块在版本 1.3.0 和 1.4.0+ 之间有所不同，因为后来的版本是异步的。用户必须像这样使用 await：

```javascript
const recast = await Recast();
```
