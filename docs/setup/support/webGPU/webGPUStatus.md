# WebGPU 状态

*请注意，我们将使用 Chrome Canary 作为 WebGPU 功能的标准浏览器，因为截至撰写本文时（2022/01/16），其他浏览器在功能支持方面仍然落后。*

## 让它工作：当前接口的状态

Babylon.js 的大部分功能现在都可以在 WebGPU 中使用。这是不起作用/部分起作用的详细列表。

### 支持不完整的功能

* 点云系统
    * WebGPU 不支持不同于 1 的 point size，因此不会考虑为 point size 设置不同于 1 的值

### 功能不工作，因为尚未实现

* 支持三角扇/线循环绘制模式
    * WebGPU 不支持这些模式，我们需要用三角带和线带来模拟它们
* 支持顶点缓冲区的浮点类型以外的类型（位置、法线、uv 等）
    * 与 WebGL 相反，在 WebGPU 中，不会自动将顶点缓冲区的类型转换为着色器使用的类型
* 处理上下文丢失/恢复
* 多视图/WebXR
    * 尚未实现，但 Chrome / WebGPU 规范也不支持

## 性能优化

最重要的优化现在已经完成（参见[优化](https://doc.babylonjs.com/setup/support/webGPU/webGPUOptimization)），其他的可以考虑：

* 从缓冲区读取数据时使用计算着色器执行一些转换
* 使用计算着色器生成 mipmap

## 其他“推荐”功能

* 使用 CreatePipelineAsync 进行异步管道创建

## 浏览器注意事项

* Chrome Canary 尚不支持所有 WebGPU 功能（或其他一些功能尚未完全发挥作用），因此这里有一些注意事项：

* 浏览器没有返回 WebGPU 功能（WebGPU capabilities）
    * 目前，我们已经为上限设置了一些硬性值
* Inspector 中的 GPU 计时不起作用，因为时间戳查询当前在 Chrome 中被禁用。如果你想启用它们，你可以使用 --disable-dawn-features=disallow_unsafe_apis 标志启动 Chrome。