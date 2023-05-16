# WebGPU 详解 - Render Pipeline 缓存

此类实现了 GPU Render Pipeline 的缓存，以避免每帧重新创建它们。

## 缓存实现

缓存是一个节点树，与bind group缓存结构相同：

````javascript
class NodeState {
    public values: { [id: number]: NodeState };
    public pipeline: GPURenderPipeline;

    constructor() {
        this.values = {};
    }
}

export class WebGPUCacheRenderPipelineTree {
    private static _Cache: NodeState = new NodeState();
    ...
}
````

不同之处在于 id 对管道状态进行编码。由于定义管道的状态有很多，因此我们需要多个 ID。目前，名单是：\

````javascript
enum StatePosition {
    StencilReadMask = 0,
    StencilWriteMask = 1,
    DepthBias = 2,
    DepthBiasSlopeScale = 3,
    DepthStencilState = 4,
    MRTAttachments1 = 5,
    MRTAttachments2 = 6,
    RasterizationState = 7,
    ColorStates = 8,
    ShaderStage = 9,
    TextureStage = 10,
    VertexState = 11, // vertex state will consume positions 11, 12, ... depending on the number of vertex inputs

    NumStates = 12
}
````

因此，第一个节点 (_Cache.values) 保存模板读取掩码值，第二个节点 (_Cache.values[stencilReadMaskValue]) 保存模板写入掩码值，第三个节点 (_Cache.values[stencilReadMaskValue].values[stencilWriteMaskValue] ) 保存深度偏置值，等等。

要在缓存中查找管道，我们只需从根节点开始，使用 stencilReadMask 当前值查找 values 属性，然后使用 stencilWriteMask 当前值查找该节点的 values 属性，依此类推，直到遍历所有状态。

## 优化

*(译者注：可以看看这里，很有意思~)*

对状态位置进行排序，以便首先列出不太可能从一条管道更改为另一条管道的状态。那是因为我们在查询缓存之前维护了一个指针（_stateDirtyLowestIndex），它包含所有已脏状态（意味着自上次管道查找以来已更改的状态）的最低索引，我们将从该索引遍历缓存而不是从 0 到查找管道。因此，_stateDirtyLowestIndex 越高，性能越好：我们将遍历更少的节点以在缓存中找到管道。

请注意，我们尝试了一个实现，其中记录在哈希映射中的渲染管道和查找键是状态值的字符串连接（请参阅 WebGPUCacheRenderPipelineString 类）但由于节点树实现速度更快（大约是当时的 2 倍）而被删除进行了比较——代码从那时起发生了变化，所以我们可能应该执行一些新的测试，但我仍然认为**节点树比哈希映射更快**）。

最后注意：values 属性是一个常规对象而不是 Map，因为在我们的测试中（使用 Chrome）**使用对象更快**。

## 监控性能

可以通过查看这些属性来评估缓存的性能（该属性应以 BABYLON.WebGPUCacheRenderPipeline 为前缀。）：

|属性|描述|
|-|-|
|NumCacheHitWithoutHash|由于自上次查找以来没有状态变化，甚至没有遍历缓存就检索渲染管道的次数：最后一个管道已返回|
|NumCacheHitWithHash|通过遍历缓存检索渲染管线的次数|
|NumCacheMiss|由于缓存中不存在而创建新渲染管线的次数|
|NumPipelineCreationLastFrame|在最后一帧期间创建的渲染管线数量 - 不应连续创建新管线，因此平均而言该值应为 0|