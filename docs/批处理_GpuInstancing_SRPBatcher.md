
## 静态批处理
将静态（不移动）游戏对象组合成大网格，并以较快的速度渲染它们。静态批处理的工作原理是将静态游戏对象转换到世界空间并为它们构建一个共享的顶点和索引缓冲区。

### 使用条件
共享相同材质并且标记为Batching Static。

### 优点
在技术上，Unity 不会减少Draw Call，而是减少渲染状态设置。

### 缺点
使用静态批处理需要额外的内存来存储组合的几何体。如果多个游戏对象在静态批处理之前共享相同几何体，则会在 Editor 中或运行时为每个游戏对象创建几何体的副本。静态批处理会导致内存和存储开销。

## 动态批处理
对于足够小的网格，此方法会在 CPU 上转换网格的顶点，将许多相似顶点组合在一起，并一次性绘制它们。

### 使用条件
* 网格不超过 300 个顶点（不超过 900 个顶点属性）。
* 不包含镜像的缩放。
* 相同材质。
* 包含相同的lightmap位置
* 多 pass 着色器会中断批处理。

### 如何开启
内置渲染管线在player设置开启。
URP渲染管线在UniversalRenderPipelineAsset开启。
HDRP渲染管线不建议开启。

### 优点
减少Draw Call。

### 缺点
会产生一些 CPU 开销。因为动态批处理的工作原理是将所有游戏对象顶点转换到 CPU 上的世界空间，所以仅在该工作小于进行绘制调用的情况下，才有优势。绘制调用的资源需求取决于许多因素，主要是使用的图形 API。例如，对于游戏主机或诸如 Apple Metal 之类的现代 API，绘制调用的开销通常低得多，通常动态批处理根本没有优势。

## GPU Instancing
使用少量draw call绘制多个相同的网格，通过减少Draw Call数量来降低CPU开销。实际上，Instancing还会在GPU上带来一些额外的开销。

### 使用条件
* 支持MeshRenderer，Graphics.DrawMesh，不支持SkinnedMeshRenderer
* 同材质同网格能合并到相同draw call
* OpenGL ES3.0及以上设备（可通过SystemInfo.supportsInstancing判断），由于驱动程序问题，对于仅具有OpenGL ES 3.0的Adreno GPU的Android设备禁用了GPU实例支持。

### 缺点
* 不支持lightmap
* 不支持culling

### 如何开启
内置shader勾选Enable Instancing
自定义shader要兼容GPU Instancing

## SRP Batcher
SRP Batcher是一个渲染循环，可加速相同着色器变体的多种材质在场景中的cpu渲染速度。SRP Batcher是一个低级渲染循环，可让Material数据保留在GPU内存中。如果Material内容不变，则SRP Batcher不需要设置并将缓冲区上载到GPU。相反，SRP Batcher使用专用的代码路径来快速更新大型GPU缓冲区中的Unity Engine属性。

### 如何开启
代码控制：GraphicsSettings.useScriptableRenderPipelineBatching = true;

### SRP Batcher兼容性
对于渲染对象的SRP Batcher代码路径：
* 呈现的对象必须是 网状或蒙皮的网格。它不能是粒子。
* 着色器必须与SRP Batcher兼容。HDRP和URP中的所有亮和不亮着色器均符合此要求（这些着色器的“粒子”版本除外）。

若要使着色器与SRP Batcher兼容：
* 您必须在一个名为“ UnityPerDraw”的CBUFFER中声明所有内置引擎属性。例如unity_ObjectToWorld，或unity_SHAr。
* 您必须在名为的单个CBUFFER中声明所有材料属性UnityPerMaterial。

## 参考
* [DrawCallBatching Manual](https://docs.unity3d.com/2019.4/Documentation/Manual/DrawCallBatching.html)
* [GPUInstancing Manual](https://docs.unity3d.com/Manual/GPUInstancing.html)
* [Unite 2016 - Unity 5.4 GPU Instancing 功能简介](http://unitytaiwan.blogspot.com/2016/05/unite-2016-unity-54-gpu-instancing.html)
* [SRPBatcher Manual](https://docs.unity.cn/2019.4/Documentation/Manual/SRPBatcher.html)
* [SRP Batcher：加速渲染](https://connect.unity.com/p/srp-batcher-jia-su-xuan-ran)
* [【直播回放】Unity 批处理/GPU Instancing/SRP Batcher](https://www.bilibili.com/video/BV1Af4y1y7Ca?from=search&seid=3448247563390578819)