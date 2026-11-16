---
title: "Vulkan Tutorial"
date: 2026-04-1T15:03:21+08:00
draft: false
---

## 入门

## 数据管理

Vulkan 在数据管理上的核心哲学，是从 OpenGL 的“隐式状态机 + 驱动全权托管”彻底转向“显式控制 + 硬件解耦”。无论承载的是网格顶点数据还是纹理像素数据，Vulkan 都把资源的逻辑声明与物理显存严格剥离：对象本身只是一份描述“我要一块什么样的存储”的元数据，真正的显存必须由开发者查询需求、显式分配并绑定上去。这一共性构成了后续所有具体类型讨论的公共前提

**缓冲对象**：

与 OpenGL 类似， Vulkan中直接管理顶点数据的对象也称为缓冲对象（Buffer Object），其用途划分与 OpenGL 高度对应——顶点、索引、统一、存储四类语义几乎一一对应，下表即为这层映射关系：

| 缓冲区用途分类 | OpenGL 绑定目标宏 | Vulkan 对应标志位 | 核心作用与说明 |
| --- | --- | --- | --- |
| 顶点缓冲区 | `GL_ARRAY_BUFFER` | `VK_BUFFER_USAGE_VERTEX_BUFFER_BIT` | 存储网格的顶点属性（如位置、法线、纹理坐标、颜色），供输入汇编阶段读取，并通过 `vkCmdBindVertexBuffers` 绑定 |
| 索引缓冲区 | `GL_ELEMENT_ARRAY_BUFFER` | `VK_BUFFER_USAGE_INDEX_BUFFER_BIT` | 存储网格的索引数据，供 `vkCmdDrawIndexed` 使用，用于复用顶点、减少内存冗余 |
| 统一缓冲区 | `GL_UNIFORM_BUFFER` | `VK_BUFFER_USAGE_UNIFORM_BUFFER_BIT` | 存储相对静态或全局的常量数据（如每帧摄像机矩阵、全局光源参数、材质常量）；容量较小，偏移需满足 `minUniformBufferOffsetAlignment` 对齐要求 |
| 着色器存储缓冲区 | `GL_SHADER_STORAGE_BUFFER` | `VK_BUFFER_USAGE_STORAGE_BUFFER_BIT` | 现代图形学与 GPU-driven 渲染的核心，容量大且支持着色器随机读写，广泛用于动态网格、粒子系统、GPU 视锥裁剪结果及 Bindless 全局资源 |

尽管二者在缓冲所定义的语义相似，但其实现机制却完全不同：

- 在 OpenGL 中，这些宏代表的是全局上下文（Context）中的绑定槽位（Binding Target）：必须先通过 glBindBuffer 把缓冲区挂载到目标槽位，后续的 glBufferData 或绘制指令才会隐式地对该槽位生效；直到 4.5 版本引入 DSA（Direct State Access）规范，才彻底改变了这一局面——通过诸如 glCreateBuffers 和 glNamedBufferData 等直接以缓冲区句柄（ID）作为参数的 API，开发者终于可以绕过全局状态机，直接对目标对象进行修改。

- 而 Vulkan 的 VkBufferUsageFlags 既不是绑定槽位，也不限制缓冲区的底层形态——它们可以通过按位或自由组合，让同一个 Buffer 既充当传输源又充当顶点缓冲。这些标志的本质是驱动的权限声明与硬件优化掩码：开发者在创建时显式告知驱动该缓冲区将参与哪些管线操作，驱动据此选择最优的显存堆分配策略、内部缓存一致性机制与校验规则。换言之，OpenGL 用标志位描述“现在绑到哪”，Vulkan 用标志位声明“将来能被怎么用”。

**纹理对象**：

在 OpenGL 中，纹理对象（Texture Object）既可以

此外， Vulkan 还继承了 OpenGL 4.5 引入的采样器和纹理对象分离的设计。也即，可以将不同采样参数封装在采样器中而无需xxx。

### 逻辑缓冲

**VKBuffer**：

****

### 图形对象

VkImage

VkImageView

VkSampler

### 帧缓冲

VkFramebuffer

### 物理显存

VkDeviceMemory

```c++
```

与 OpenGL 这种由驱动在后台帮你动态扩容、隐式管理的黑盒机制不同，Vulkan 的显存管理完全是显式（Explicit）的。当你向 GPU 申请一块物理显存（VkDeviceMemory）时，你必须像在 C 语言中使用 malloc 一样，清清楚楚地告诉驱动

**查询限制**：

**AMD 扩展函数**：

## 命令系统

**即时渲染模式**：

OpenGL 的绘制模型是即时模式（Immediate Mode）：每一次 glDraw* 调用都会立即进入驱动，驱动当场解析当前状态机、做合法性校验、翻译成硬件寄存器写入，然后交给 GPU。这种机制将 CPU 与 GPU 的调用节奏强耦合在一起——CPU 每发出一条命令，驱动就要立刻介入一次，导致整个系统的吞吐量极易被锁死在最低效的环节。

Vulkan 则彻底抛弃了这一模式，改为“先录制、后提交”的延迟执行范式：CPU 只负责将渲染指令序列编码进命令缓冲区，而真正提交给硬件的时机完全由开发者掌控。这彻底解耦了命令生成与实际执行的时间线，不仅允许 CPU 提前准备好未来数帧的工作负载，还为多线程并行录制铺平了道路。

**有限状态机**:

为了理解为什么即时模式下的有限状态机（Finite State Machine）会引发严重的性能崩塌，我们需要从硬件底层和驱动两方面来剖析：

- 硬件等待（Pipeline Bubbles）：与依赖复杂分支预测和低延迟的 CPU 不同，GPU 通过 SIMT 架构追求极致吞吐。当渲染管线正在高速处理数据时，如果频繁更改状态，硬件被迫进行流水线排空（Pipeline Flushing），等待旧任务清空；同时，底层的固定功能硬件（如 ROP 渲染输出单元、光栅化器）必须重新配置物理电路，这会产生严重的硬件气泡和算力浪费。

- 驱动验证（Driver Overhead）：在传统 API 下，每次调用绘制命令或切换状态时，CPU 端的图形驱动程序必须在后台进行海量的动态验证。驱动需要实时检查当前的所有状态、纹理绑定和着色器是否冲突，并动态翻译成硬件能懂的寄存器指令。

这种双重压力（CPU 驱动验证开销 + GPU 硬件流水线重建代价）构成了传统 API 挥之不去的性能瓶颈。

### 命令缓冲与命令池

**命令缓冲**：

VkCommandBuffer 是一个纯粹的 CPU 端内存容器，记录的是供驱动直接解释或转换为底层硬件指令的二进制字节流。根据其作用范围与提交方式，它可分为两大类型：

- 主命令缓冲区（Primary Command Buffer）：拥有完整的生命周期管理能力，可以直接提交给硬件队列（VkQueue）执行。同时，它也可以作为主干，在执行过程中嵌套调用次级命令缓冲区。

- 次级命令缓冲区（Secondary Command Buffer）：不能直接提交给硬件队列，必须被主命令缓冲区通过 vkCmdExecuteCommands 显式调用。它的核心架构意义在于支持多线程并行录制——不同的 CPU 渲染线程可以各自独立、无锁地录制属于自己的次级命令缓冲区，最后由主线程统一组装到主命令缓冲区中，这是现代多核引擎压榨 CPU 录制性能的关键支撑。

**命令缓冲池**：

由于命令缓冲区频繁创建和销毁会带来巨大的内存分配开销，Vulkan 引入了命令池。所有 VkCommandBuffer 必须从指定的 VkCommandPool 中分配。

值得注意的是，命令池与从它分配的命令缓冲区在多线程环境下不具备隐式线程安全性。通常的做法是：每个 CPU 渲染线程拥有自己独立的 VkCommandPool，从而实现真正的无锁化、多线程并行录制。

### 命令录制

命令录制是把高层渲染意图编码为指令流的过程，其生命周期由一对边界函数严格界定：vkBeginCommandBuffer 使命令缓冲进入 Recording 状态，vkEndCommandBuffer 使其进入可执行状态，两者之间的所有 vkCmd* 调用都会被顺序记录下来。

```c++
vkBeginCommandBuffer(cmd, &beginInfo);
// 1.
vkCmdBeginRenderPass(cmd, &renderPassInfo, VK_SUBPASS_CONTENTS_INLINE);
// 2. 绑定渲染管线 PSO
vkCmdBindPipeline(cmd, VK_PIPELINE_BIND_POINT_GRAPHICS, pipeline);
// 3. 绑定资源描述符
vkCmdBindDescriptorSets(cmd, ...);
vkCmdBindVertexBuffers(cmd, ...);
vkCmdSetViewport(cmd, ...);
vkCmdSetScissor(cmd, ...);
vkCmdDraw(cmd, vertexCount, instanceCount, 0, 0);
vkCmdEndRenderPass(cmd);
vkEndCommandBuffer(cmd);
```

### 命令提交

**硬件队列**：

物理 GPU 通常包含多种不同的硬件引擎（如图形引擎、计算引擎、DMA 传输引擎）。Vulkan 将其抽象为不同类型的 VkQueue（如 Graphics Queue、Compute Queue、Transfer Queue）。开发者通过 vkQueueSubmit 将一个或多个主命令缓冲区打包成批次（Batch），投递到对应的硬件队列中。

**显式同步机制**：

与 OpenGL 在后台默默处理同步、频繁导致 CPU 陷入隐式阻塞的机制不同，Vulkan 将所有的同步责任完全交给了开发者。在提交命令时，必须通过显式的同步原语来管理复杂的硬件依赖：

- VkSemaphore（信号量）：用于在同一个 GPU 的不同队列之间，或者 GPU 与呈现引擎（Swapchain）之间建立硬件级执行依赖（例如确保图像可用后再开始渲染）。
- VkFence（栅栏）：用于让 CPU 能够精确感知某一批次任务是否已经在 GPU 上彻底执行完毕，从而安全地回收或重用上一帧的内存与资源。

## 管线状态

**Fixed-function state**:

Draw Call 触发的状态切换，主要指光栅化器、ROP、混合单元、深度/模板测试单元、着色器绑定、纹理绑定、顶点输入布局等固定功能或管线状态的变更。这和 CPU 多线程中切换上下文的方式完全不同：

- CPU 切换线程时需要保存和恢复寄存器、栈指针、程序计数器等线程上下文；
- 而 GPU 切换 Draw Call 时并不是为每个 Draw Call 保存完整线程上下文，而是要保证整条渲染管线的配置与新绘制任务一致。

因此，GPU 中 Draw Call 切换的本质是固定功能模块和管线状态的重新配置，而非线程上下文的替换。

### 顶点数据格式

### 管线固定功能

## 资源绑定
