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

## 传送

你可以使用以下调用将代理传送到任何位置：

```javascript
crowd.agentTeleport(agentIndex, navigationPlugin.getClosestPoint(destinationPoint));
```

请注意，传送时导航状态会被重置。你需要调用 agentGoto 来选择一个新的目的地。

## 代理方向和下一个路径目标

Recastjs 人群系统不处理代理方向。但速度是可用的，可以根据速度方向调整几何体的方向。为此，你需要使用 Math.atan2, 如下例所示。请注意速度向量的长度。如果它不够大，可能会遇到抖动问题。

```javascript
let velocity = crowd.getAgentVelocity(agentIndex);
if (velocity.length() > 0.2) {
  const desiredRotation = Math.atan2(velocity.x, velocity.z);
  // interpolate the rotation on Y to get a smoother orientation change
  ag.mesh.rotation.y = ag.mesh.rotation.y + (desiredRotation - ag.mesh.rotation.y) * 0.05;
}
```

[代理方向和下一个路径目标](https://playground.babylonjs.com/#6AE0RP)

代理的立方体根据速度进行定向，一个灰色的小方块放置在下一个路径拐角的位置。

## 代理到达目标观察者

当代理到达目的地（即在目的地半径内）时，会自动触发一个观察对象。默认情况下，半径是代理的半径，但可以使用 IAgentParameters 对象中的 reachRadius 数字属性进行更改。如果人群中有太多代理试图到达同一目的地，可能会发生瓶颈，少数代理会到达目的地。请确保正确设置这些值。要添加一个可观察对象，只需添加你的函数：

```javascript
const crowd = navigationPlugin.createCrowd(10, 0.1, scene);
...
crowd.onReachTargetObservable.add((agentInfos) => {
    console.log("agent reach destination: ", agentInfos.agentIndex);
});
```
