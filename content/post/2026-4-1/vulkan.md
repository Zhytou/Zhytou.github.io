---
title: "Vulkan Tutorial"
date: 2026-04-1T15:03:21+08:00
draft: false
---

- [数据管理](#数据管理)
  - [逻辑缓冲](#逻辑缓冲)
  - [图形对象](#图形对象)
  - [帧缓冲](#帧缓冲)
  - [物理显存](#物理显存)
- [资源描述符](#资源描述符)
  - [绑定点](#绑定点)
  - [分组槽](#分组槽)
  - [无分组](#无分组)
- [命令系统](#命令系统)
  - [命令缓冲与命令池](#命令缓冲与命令池)
  - [命令录制](#命令录制)
  - [命令提交](#命令提交)
- [管线状态](#管线状态)
  - [顶点数据格式](#顶点数据格式)
  - [管线固定功能](#管线固定功能)

## 数据管理

Vulkan 在数据管理上的核心哲学，是从 OpenGL 的“隐式状态机 + 驱动全权托管”彻底转向“显式控制 + 硬件解耦”。无论承载的是网格顶点数据还是纹理像素数据，Vulkan 都把资源的逻辑声明与物理显存严格剥离。这使得开发者总是先创建一个描述资源用途、尺寸、格式和访问方式的 Vulkan 对象，再通过显式的内存分配与绑定，将其映射到实际的 GPU 内存。

这套设计的核心是两个层次：

- 逻辑资源（Resource Object）：描述 GPU 要使用什么数据、数据的组织方式以及使用场景。例如 VkBuffer、VkImage。
- 物理内存（Memory Allocation）：负责实际承载数据，决定数据存放在哪里、如何被 GPU 访问。例如 VkDeviceMemory。

值得注意的是，二者之间并非简单的一一对应关系：

- 一个 VkDeviceMemory 可以被多个资源分别绑定到不同、不重叠的内存区域。
- 一个 VkBuffer 或 VkImage 可以绑定到一块更大的 VkDeviceMemory 中，而不必独占整块内存。

换句话说，资源的逻辑尺寸与实际内存分配尺寸也不一定相同，实际分配需要考虑内存对齐、资源布局、硬件粒度等因素。

### 逻辑缓冲

与 OpenGL 类似，Vulkan 中直接管理顶点数据的对象也称为缓冲对象（Buffer Object），其用途划分与 OpenGL 高度对应——顶点、索引、统一、存储四类语义几乎一一对应。

| 缓冲区用途分类 | OpenGL 绑定目标宏 | Vulkan 对应标志位 | 核心作用与说明 |
| --- | --- | --- | --- |
| 顶点缓冲区 | `GL_ARRAY_BUFFER` | `VK_BUFFER_USAGE_VERTEX_BUFFER_BIT` | 存储网格的顶点属性（如位置、法线、纹理坐标、颜色），供输入汇编阶段读取，并通过 `vkCmdBindVertexBuffers` 绑定 |
| 索引缓冲区 | `GL_ELEMENT_ARRAY_BUFFER` | `VK_BUFFER_USAGE_INDEX_BUFFER_BIT` | 存储网格的索引数据，供 `vkCmdDrawIndexed` 使用，用于复用顶点、减少内存冗余 |
| 统一缓冲区 | `GL_UNIFORM_BUFFER` | `VK_BUFFER_USAGE_UNIFORM_BUFFER_BIT` | 存储相对静态或全局的常量数据（如每帧摄像机矩阵、全局光源参数、材质常量）；容量较小，偏移需满足 `minUniformBufferOffsetAlignment` 对齐要求 |
| 着色器存储缓冲区 | `GL_SHADER_STORAGE_BUFFER` | `VK_BUFFER_USAGE_STORAGE_BUFFER_BIT` | 现代图形学与 GPU-driven 渲染的核心，容量大且支持着色器随机读写，广泛用于动态网格、粒子系统、GPU 视锥裁剪结果及 Bindless 全局资源 |

尽管二者在缓冲所定义的语义相似，但其实现机制却完全不同：

- OpenGL 的绑定目标：代表全局 Context 中的状态槽位。通过 glBindBuffer 将 Buffer 绑定到某个目标，后续操作会隐式作用于当前绑定对象。
- Vulkan 的 Usage Flags：代表资源创建时声明的使用权限和用途。它既不是绑定槽位，也不意味着资源会自动绑定到某个硬件单元。

例如：

```c++
VkBufferCreateInfo bufferInfo{};
bufferInfo.sType = VK_STRUCTURE_TYPE_BUFFER_CREATE_INFO;
bufferInfo.size = bufferSize;
bufferInfo.usage =
    VK_BUFFER_USAGE_VERTEX_BUFFER_BIT |
    VK_BUFFER_USAGE_TRANSFER_DST_BIT;
bufferInfo.sharingMode = VK_SHARING_MODE_EXCLUSIVE;

vkCreateBuffer(
    device,
    &bufferInfo,
    nullptr,
    &vertexBuffer
);
```

这里声明该 Buffer 可以作为顶点缓冲区使用，同时也可以作为数据传输的目标。需要注意，Usage Flags 必须在创建资源时声明，不能像 OpenGL 那样通过后续改变绑定目标来改变资源用途。

另外，与 OpenGL 中通过 VAO 描述顶点数据布局类似，Vulkan 也需要单独描述顶点数据的格式、偏移和步长。不过，Vulkan 并没有完全对应 OpenGL VAO 的对象，而是通过 Pipeline 的 Vertex Input State 描述顶点输入布局，并在绘制命令中通过 vkCmdBindVertexBuffers 绑定实际的顶点缓冲区。

因此，VkBuffer 本身只负责承载线性数据，并不关心这些数据究竟代表位置、法线还是纹理坐标。

### 图形对象

在 OpenGL 中，纹理对象（Texture Object）既承载图像数据，也承担图像格式、尺寸、Mip Level 等资源属性的描述。不过，随着 OpenGL 版本的发展，纹理资源与采样状态逐渐实现分离。Vulkan 延续并进一步明确了这种设计，将原本耦合在一起的功能拆分为三个独立对象：

- VkImage：描述图像资源本身，包括尺寸、格式、Mip Level、Array Layer、Sample Count 以及使用方式。
- VkImageView：描述如何访问和解释某个 Image，可以选择特定的 Mip Level、Array Layer 和图像格式。
- VkSampler：描述采样行为，例如过滤方式、寻址模式和各向异性过滤。

### 帧缓冲

### 物理显存

与 OpenGL 这种由驱动在后台帮你动态扩容、隐式管理的黑盒机制不同，Vulkan 的显存管理完全是显式（Explicit）的。当你向 GPU 申请一块物理显存（VkDeviceMemory）时，你必须像在 C 语言中使用 malloc 一样，清清楚楚地告诉驱动

**资源限制查询**：

Vulkan 不会假设所有 GPU 都拥有相同的内存结构、资源大小限制和对齐要求。

因此，开发者需要在初始化阶段查询物理设备的能力和限制。常用接口包括：

- vkGetPhysicalDeviceProperties：查询物理设备的基本属性及设备限制。
- vkGetPhysicalDeviceMemoryProperties：查询 Memory Heap 和 Memory Type。
- vkGetPhysicalDeviceFormatProperties：查询特定图像格式支持的功能。
- vkGetPhysicalDeviceImageFormatProperties：查询特定 Image 创建参数组合是否受支持。

其中，VkPhysicalDeviceLimits 中与资源管理密切相关的字段包括：

**分配与映射**：

****

**管理组件**：

## 资源描述符

在现代图形管线中，着色器需要读取外部的缓冲区（UBO / SSBO）或纹理（Texture / Sampler）数据。然而，GPU 硬件核心在执行着色器指令时，无法直接理解 C++ 端抽象的 VkBuffer 或 VkImage 句柄。

描述符（Descriptor）正是连接这两者的底层桥梁——它本质上是一块对 GPU 硬件指令集直接可读的“胖指针（Fat Pointer）”，内部封装了资源在显存中的物理起始地址、字节长度、像素格式以及采样寻址规则等硬件元数据。

> 瘦指针（Thin Pointer / 裸指针）：float* ptr —— 只有 8 个字节的起始地址。硬件只知道“从哪开始读”，但不知道读到哪算越界、也不知道里面存的是什么格式。
> 胖指针（Fat Pointer / 带元信息的指针）内部不仅存了实际存放数据的显存物理基地址（Base Address），还硬编码了硬件执行单元直接需要的访问边界与格式说明。

### 绑定点

绑定点（Binding）是 Shader 与 CPU 端建立资源连接的最小逻辑槽位。

以 OpenGL 为例，GLSL 通过 binding 关键字声明了资源槽位。此时，C++ 端只需 glBindBufferBase 或 glActiveTexture + glBindTexture 便可将资源正确链接。

```glsl
layout(std140, binding = 0) uniform CameraBlock {
    mat4 uViewMatrix;
    mat4 uProjMatrix;
    mat4 uInvViewMatrix;
    mat4 uInvProjMatrix;
    vec3 uCameraPos;
    float uCameraType;
    float uFov;
    float uNear;
    float uFar;
    float uAspect;
};

layout(binding = 0) uniform sampler2D tAlbedoMap;
layout(binding = 1) uniform sampler2D tNormalMap;
layout(binding = 2) uniform sampler2D tMRAOMap;
```

**基于全局状态机隐式资源绑定**：

尽管 OpenGL 中也有资源绑定的概念，但它完全建立在隐式的全局状态机之上，这在现代高性能渲染架构下暴露出三个核心局限：

- 槽位命名空间割裂且状态易受污染：OpenGL 为不同类型的资源（UBO、SSBO、Sampler和Texture）维护着各自独立的全局绑定表（因此上例中 CameraBlock 和 tAlbedoMap 均可使用 binding = 0）。然而，这种依赖全局上下文的状态极易在复杂的渲染 Pass 切换中被意外覆盖，且无法在多线程下并发录制。
- 运行时驱动验证开销大：在执行 glDraw* 之前，驱动程序必须在后台动态检查当前全局状态槽位上的资源类型、格式是否与活跃 Shader 匹配，导致大量不可预测的 CPU 驱动验证开销。
- 零散绑定带来的调用碎片化：每切换一个物体或材质，往往需要多次调用 glActiveTexture 和 glBindTexture 逐一刷新各个槽位。

**基于对象状态的显式资源绑定**：

为了彻底解决这些问题，Vulkan 将传统的全局隐式绑定重构为基于对象（Object-Based）的显式资源绑定体系：

- 统一且显式的槽位镜像：在 Vulkan 中，所有资源共享同一套扁平化的 binding 编号空间，不允许重复。C++ 端不再依赖隐式状态匹配，而是必须通过 VkDescriptorSetLayoutBinding 结构体，在创建管线前一比一显式声明每个 binding 的编号、资源类型及可见的着色器阶段。
- 编译期接口锁定：驱动在创建管线时即根据这些显式声明完成硬件寄存器映射与合法性校验，将运行时开销前置。

**描述符集布局绑定点**：

VkDescriptorSetLayoutBinding 是 Vulkan 中描述符接口的最小配置单元，它精确描述了单个绑定槽位的结构契约，包括：

- 绑定点编号（binding）：与 Shader 中 layout(binding = X) 严格对应的槽位索引。
- 资源类型（descriptorType）：指定该槽位接收的是 UBO、SSBO 还是组合图像采样器等。
- 描述符数量（descriptorCount）：支持单个资源或资源数组（阵列大小）。
- 着色器阶段可见性（stageFlags）：限定仅对特定阶段（如顶点或片元着色器）开放访问，协助驱动优化硬件寄存器分配。

其创建方式如下：

```c++
VkDescriptorSetLayoutBinding cameraBinding{};
cameraBinding.binding            = 0;                                         // 对应 GLSL 的 binding = 0
cameraBinding.descriptorType     = VK_DESCRIPTOR_TYPE_UNIFORM_BUFFER;         // 资源类型：UBO
cameraBinding.descriptorCount    = 1;                                         // 数量：1个（非数组）
cameraBinding.stageFlags         = VK_SHADER_STAGE_VERTEX_BIT;                // 仅在顶点着色器可见
cameraBinding.pImmutableSamplers = nullptr;                                   // 采样器专用，普通情况填 nullptr
```

### 分组槽

仅仅拥有统一的绑定点编号还不足以解决高频切换带来的 CPU 驱动开销。如果每次切换材质都要重新绑定，渲染循环依然会陷入调用碎片化。为此，Vulkan 引入了“描述符集布局（Layout）与描述符集实例（Set）分离”的双层分组体系，实现了资源的批量打包与原子级切换。

**描述符集布局**:

VkDescriptorSetLayout 实际上是由一组逻辑相关的 VkDescriptorSetLayoutBinding 组装而成，本质上是一份纯粹的接口类型签名。

- 与具体数据彻底解耦：它只定义“第几号槽位放什么类型的资源”，不包含任何真实的 VkBuffer 或 VkImageView 句柄。
- 服务于管线编译：在创建渲染管线时，只需将若干个 VkDescriptorSetLayout 装配进 VkPipelineLayout 交给驱动，驱动即可提前锁定所有资源的硬件寄存器布局。这使得同一条编译好的渲染管线，可以无缝复用于成千上万种拥有不同贴图和参数的材质实例。

**描述符集和描述符池**：

如果 VkDescriptorSetLayout 是接口，那么 VkDescriptorSet 就是这个接口的一个实例化对象，而 VkDescriptorPool 则是负责批量分配这些容器的底层内存池：

- 池化预分配（VkDescriptorPool）：为了避免在渲染循环中频繁向操作系统申请零散内存，Vulkan 要求预先创建一个指定容量上限的 VkDescriptorPool（声明池子最多能容纳多少个 Set 以及各类描述符的总数），随后再从池中高速分配出具体的 VkDescriptorSet。
- 资源加载期的元数据写入（vkUpdateDescriptorSets）：刚分配出来的 VkDescriptorSet 内部为空。C++ 端在资源加载阶段仅执行一次，通过 vkUpdateDescriptorSets 将具体的 VkBuffer（经 VkDescriptorBufferInfo 包装）和 VkImageView + VkSampler（经 VkDescriptorImageInfo 包装）翻译成硬件胖指针，永久写入对应的槽位中。此后在渲染循环中绝不再调用该函数。
- 按更新频率分层的插槽复用：在创建 VkPipelineLayout 时，多个 Layout 会按顺序映射到着色器中的 set = 0, 1, 2... 槽位。引擎通常按资源的更新频率进行分层规划（例如将每帧一次的全局数据固定在 set = 0，将按材质切换的贴图组绑定在 set = 1）。
- 渲染执行期的原子级批量绑定（vkCmdBindDescriptorSets）：进入每帧的命令缓冲区录制循环时，着色器中声明的 set 槽位编号是永久固定的。切换材质时，只需调用一次 vkCmdBindDescriptorSets，将新材质的 VkDescriptorSet 实例指针替换进固定的槽位中（例如 firstSet = 1）。由于硬件元数据早已在加载期烘焙完毕，该操作在底层仅相当于更新显存基地址指针（纳秒级），彻底避开了运行时的合法性校验与描述符重组。

### 无分组

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

**固定管线状态**:

Draw Call 触发的状态切换，主要指光栅化器、ROP、混合单元、深度/模板测试单元、着色器绑定、纹理绑定、顶点输入布局等固定功能或管线状态的变更。这和 CPU 多线程中切换上下文的方式完全不同：

- CPU 切换线程时需要保存和恢复寄存器、栈指针、程序计数器等线程上下文；
- 而 GPU 切换 Draw Call 时并不是为每个 Draw Call 保存完整线程上下文，而是要保证整条渲染管线的配置与新绘制任务一致。

因此，GPU 中 Draw Call 切换的本质是固定功能模块和管线状态的重新配置，而非线程上下文的替换。

### 顶点数据格式

### 管线固定功能
