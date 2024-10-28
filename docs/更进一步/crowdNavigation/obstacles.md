# 添加和移除障碍物

## 前提条件

在内部，支持障碍物的导航网格与“标准”用例中的静态导航网格有所不同。障碍物将标记瓦片为脏的。这些瓦片需要重新处理以计算导航网格的一部分。为此，Recast 引入了瓦片的概念。每个瓦片是导航网格的一个方形部分。瓦片在每一帧中处理。这样可以在几帧内处理巨大的变化。

为了构建一个瓦片化的导航网格而不是单个网格，请在导航网格创建参数中添加以下几行：

```javascript
const navmeshParameters = {
  cs: 0.2,
  ch: 0.2,
  walkableSlopeAngle: 0,
  walkableHeight: 0.0,
  walkableClimb: 0,
  walkableRadius: 1,
  maxEdgeLen: 12,
  maxSimplificationError: 1.3,
  minRegionArea: 8,
  mergeRegionArea: 20,
  maxVertsPerPoly: 6,
  detailSampleDist: 6,
  detailSampleMaxError: 15,
  borderSize: 1,
  tileSize: 20,
};
```

注意添加 tileSize。它是每个瓦片的世界单位大小。如果它不存在或值为零，则会回退到标准用法，障碍物将不起作用。此外，根据你的用例，这个值必须仔细选择，以在瓦片数量过多和 CPU 密集型更新之间进行权衡。

注意：ComputePath 方法将在实例化人群时专门处理障碍物。否则，路径将遵循导航网格而不考虑障碍物。如果你的用例涉及计算带有障碍物的路径，代理是可选的，但人群是必需的。

## 障碍物API

一旦导航网格更新为考虑瓦片，障碍物可以通过三个简单的函数进行访问：

```javascript
addCylinderObstacle(position: Vector3, radius: number, height: number): IObstacle;
addBoxObstacle(position: Vector3, extent: Vector3, angle: number): IObstacle;
removeObstacle(obstacle: IObstacle): void;
```

保留一个已添加障碍物的列表，以便稍后移除它们。与导航网格不碰撞的障碍物将没有影响。

[添加门到一个导航网格](https://playground.babylonjs.com/#WCSDE1)