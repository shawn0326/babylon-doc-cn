
入门 - 第二章 - Web应用布局
===

## 多样化的页面布局

虽然您可能希望游戏或应用程序占据页面的大部分，但仍然可能需要一些空间作为说明部分。只需将您的<canvas>元素放在<div>中，然后按照实际需要排列这些元素。

````html
<div id = "holder">
        <canvas id="renderCanvas" touch-action="none"></canvas> <!-- touch-action="none" for best results from PEP -->
</div>
<div id = "instructions">
    <br/>
    <h2>Instructions</h2>
    <br/>
    Instructions Instructions Instructions Instructions Instructions 
    Instructions Instructions Instructions Instructions Instructions 
</div>
````

同时设置一些样式

````html
<style>
    #holder {
        width: 80%;
        height: 100%;
        float: left;
    }

    #instructions {
        width: 20%;
        height: 100%;
        float: left;
        background-color: grey;
    }
</style>
````

[示例应用和说明](https://doc.babylonjs.com/webpages/app3.html)引入村庄模型

当然你也可以纯使用代码搭建场景

[示例应用和说明](https://doc.babylonjs.com/webpages/app4.html)使用代码搭建村庄

在接下来的教程中我们将通过加入一些带动画的车辆来为场景加入一些动态元素。车子的轮胎需要独立于车身而旋转。为了达到这个目的，需要将轮子添加到车身上。