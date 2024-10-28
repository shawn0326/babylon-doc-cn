# 群代理

## 群和导航代理

现在我们有了导航网格，可以创建自主代理并让它们在该导航网格约束内导航。代理将找到到达目的地的最佳路径，同时避开其他人群代理。代理附加到一个变换（Transform）。这意味着你需要附加一个网格来看到它们，但也可以附加几乎任何东西。

演示可以在以下地址找到：[群和导航代理](https://playground.babylonjs.com/#X5XCVT#240)

点击导航网格上的任意位置，使代理前往该位置。

## 如何使用它？ 

首先要创建一个所有代理都属于的群。参数包括群中的最大代理数量、最大代理半径和场景。

```javascript
const crowd = navigationPlugin.createCrowd(10, 0.1, scene);
```

然后要创建一个代理并将其附加到一个变换，调用：

```javascript
const agentIndex = crowd.addAgent(position, agentParameters, transform);
```

就是这样！你将得到一个不移动的代理。现在我们想要移动它。

```javascript
crowd.agentGoto(agentIndex, navigationPlugin.getClosestPoint(endPoint));
```

此代码将获取导航网格上最接近 endPoint 的位置。然后它会要求代理前往该位置。根据你的代理参数，它到达那里的速度可能会快或慢。

## 代理参数

radius - 代理的半径。世界单位。

height - 高度。世界单位。

maxAcceleration - 最大加速度。每秒世界单位。

maxSpeed - 最大速度。每秒世界单位。

collisionQueryRange - 代理碰撞系统将在该半径内处理其他代理。世界单位。

pathOptimizationRange - 路径将如何优化并变得更直。

separationWeight - 系统将多努力地尝试分离代理。值为 0 表示它不会尝试，代理可能会碰撞。

你可以通过调用以下方法来更新每个代理的任何这些参数：

```javascript
// change speed and max speed
crowd.updateAgentParameters(agentIndex, { maxSpeed: 10, maxAcceleration: 200 });
```

TODO
