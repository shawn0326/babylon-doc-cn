# WebGPU 详解 - BindGroup缓存

此类实现了 GPU BindGroup 的缓存，以避免每帧重新创建它们。

## 缓存实现

缓存是一个节点树：

````javascript
class WebGPUBindGroupCacheNode {
    public values: { [id: number]: WebGPUBindGroupCacheNode };
    public bindGroups: GPUBindGroup[];

    constructor() {
        this.values = {};
    }
}

export class WebGPUCacheBindGroups {
    private static _Cache: WebGPUBindGroupCacheNode = new WebGPUBindGroupCacheNode();
    ...
}
````

值对象中的 id 键是BindGroup资源的 id：uniform/storage 缓冲区、采样器或纹理。uniform/storage 缓冲区和纹理的 id 值只是相应类的 uniqueId 属性（分别为 DataBuffer.uniqueId 和 InternalTexture.uniqueId / ExternalTexture.uniqueId）。对于采样器，它是采样器哈希码（由 WebGPUCacheSampler.GetSamplerHashCode() 计算）。缓存是通过遍历着色器（封装在 WebGPUPipelineContext 中）使用的所有缓冲区/采样器/纹理按此顺序遍历/构建的。

## 实现的限制

资源的位置（[[group(G), binding(B)]] 语法中的组和绑定索引）不在 id 中，并且 id 不是全局唯一的（因为它们不是来自同一个池：缓冲区和纹理 uniqueId 属性有一个单独的池），因此从理论上讲，当两组资源指向同一缓存条目时，可能会发生一些冲突。实际上，它可能永远不会发生。使缓存万无一失意味着让它变得更慢，并且 WebGPU 实现已经因为必须处理某些对象的缓存而遭受很多损失（主要是绑定组和渲染管道）......

另请注意，所有uniform缓冲区在 Babylon 中的偏移量均为 0，我们没有用例将相同的缓冲区用于不同的容量值：这意味着我们不需要考虑偏移量/大小缓存中的缓冲区，只有 id。

## 优化

有一个缓存优化，如果自上次缓存查询以来绘图和材质上下文没有改变，我们只需返回现有的绑定组。实际上，绘制上下文包含 uniform/storage 缓冲区的列表，而材质上下文包含着色器使用的纹理和采样器的列表：如果这些列表没有更改，则之前创建的绑定组仍然有效。

## 监控性能

可以通过查看这些属性来评估缓存的性能（该属性应以 BABYLON.WebGPUCacheBindGroups 为前缀。）

|属性|描述|
|-|-|
|NumBindGroupsCreatedTotal|自程序启动以来创建的绑定组总数|
|NumBindGroupsCreatedLastFrame|在最后一帧期间创建的绑定组数 - 为了获得最佳缓存使用率，该值平均应为 0|
|NumBindGroupsLookupLastFrame|遍历缓存检索到的绑定组数|
|NumBindGroupsNoLookupLastFrame|由于自上次对此着色器的缓存查询后缓冲区/纹理/采样器未发生任何更改而未遍历缓存而检索到的绑定组数|