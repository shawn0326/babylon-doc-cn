# WebGPU 杂项优化

这里列出了一些优化 WebGPU 速度的杂项技巧。

## 避免每帧创建太多资源

如果调用 engine.enableEffect()，请确保将 DrawWrapper 传递给它而不是 Effect。否则每帧都会创建一些 WebGPU 资源。

要检查您是否没有创建不必要的资源，当您的应用程序正在运行并且“稳定”（意味着您没有在最后一帧中创建新对象）时，请检查 engine.countersLastFrame 并确保 numEnableEffects 为 0。调用时 numEnableEffects > 0 engine.enableEffect() 带有 Effect 而不是 DrawWrapper：只有 numEnableDrawWrapper 应该是非 0。

## 优化后处理

如果在 onApply 观察器中手动设置 textureSampler 属性，请为后处理设置 externalTextureSamplerBinding = true 以优化性能。

如果可能，不要将 PostProcess 的构造函数的 reusable 参数设置为 true。否则，用作后期处理渲染目标的两个纹理之间会不断交换，这在非兼容模式下尤其糟糕，因为缓存渲染包将在每一帧中重新创建。