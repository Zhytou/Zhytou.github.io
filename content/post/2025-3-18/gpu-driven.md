---
title: "GPU-Driven Pipeline"
date: 2025-03-18T19:46:08+08:00
draft: False
---


GPU驱动管线（GPU-Driven Pipeline）是一种将原本由CPU承担的大量渲染任务转移到GPU上进行处理的渲染管线架构。在传统渲染管线中，CPU负责管理场景中的大量数据，如模型的加载、裁剪、排序以及绘制调用的提交等，这使得CPU成为渲染流程中的瓶颈。而GPU驱动管线则通过Indirect Draw、Compute Shader等机制让GPU能够更自主地处理这些任务，充分发挥其强大的并行计算能力，从而突破了渲染性能。

## Bottleneck of Traditional Rendering Pipeline

1. **High CPU overload**

- 视锥/遮挡裁剪
- 准备drawcall

2. **CPU can not follow up GPU**

3. **GPU state exchange overhead when solving large amount of drawcalls**

## Key Ideas of GPU-Driven Pipeline

### Indirect Draw

**Draw from GPU buffer**

GPU驱动管线的核心目标是最小化CPU参与渲染流程的频率。传统绘制方式（如 glDrawArray/glDrawElements）需由CPU直接传递绘制参数（如图元数量、索引范围等），这会导致CPU-GPU通信开销较高。间接绘制则将这些参数存储在GPU可访问的缓冲区中，由GPU自行读取参数并执行绘制，从而大幅减少CPU的介入。

**Api support**

```c++
// CPU直接传递参数
glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, 0); 

// 参数来自GPU缓冲区
glDrawElementsIndirect(GL_TRIANGLES, GL_UNSIGNED_INT, buffer_offset);
```

### Compute Shader

**High-speed general purpose computing**

为了动态生成间接绘制的参数，现代渲染管线又引入了另一项关键技术，也即通用计算着色器Compute Shader。与传统的Vertex Shader和Fragment Shader不同，Compute shader不直接参与图形渲染，而是专注于利用GPU的大规模并行计算能力执行非图形渲染任务。比如，根据场景数据来决定实际需要绘制的图元数量，从而避免不必要的渲染操作。

**Key Features**

- Parallel Processing：Compute Shader可以同时处理数千个线程，极大地提高了数据处理效率。
- Shared Memory：支持线程组内的快速数据共享，减少全局内存访问。
- Barriers：提供线程同步机制，确保数据一致性。
- Atomic Operations：支持原子操作，如递增、比较和交换等。

**Typical Use Cases**

- Tessellation Control：根据 LOD 需求动态生成网格细分级别。
- Culling：执行视锥体剔除、遮挡剔除等操作，筛选可见物体。
- Physics Simulation：实现流体模拟、布料模拟等物理效果。

### GPU-Driven Culling

裁剪（Culling）是根据观察区域确定图形对象可见部分并移除不可见部分的过程。它通过减少冗余数据处理，显著提升渲染效率。

**Conventional culling**

在传统渲染管线中，可用于裁剪的时间点包括：

- 加载或更新模型时；
- 光栅化之间；
- 光栅化之后片元着色之前。

其中，将数据传入顶点着色器之前的裁剪任务由CPU驱动，它包括：

- 细节剔除（Detail Culling）：根据物体与相机的距离，选择不同细节程度的模型进行渲染。这一过程中，CPU负责根据物体的距离和其他条件来选择合适的层次细节（Level of Detail, LoD）模型，然后将其传递给GPU进行渲染。
- 遮挡裁剪（Soft Occlusion Culling）：预处理判断物体遮挡关系，并以剔除。这一操作可以在CPU端进行预处理，通过一些算法（如八叉树、层次包围盒等）来快速判断，也可以在GPU端使用硬件加速算法如Early-Z等来实现。

而由GPU硬件完成的裁剪则包括：

- 视锥剔除（Frustum Culling）：检测物体是否位于视锥体内，直接剔除或裁剪视锥体外部的图元。
- 背面剔除（Backface Culling）：基于三角形面的法线方向（如右手定则）剔除不可见的背面图元，减少无效渲染计算。

**Compute based culling**

随着GPU计算能力的提升，GPU-Driven Culling技术逐渐成为主流。它将裁剪任务从CPU转移到GPU上进行处理，充分发挥GPU的并行计算能力，提升渲染效率。具体而言，它常常和间接绘制参数生成的逻辑放在一起，由Compute Shader一并实现。

### Mesh Cluster Rendering

**Mesh clustering**

通过将场景中具有​​相似渲染属性​​（如材质、着色器、纹理、渲染状态等）的网格对象动态划分为若干​​集群（Cluster）​​，每个集群内的子网格（Sub-Mesh）共享相同的渲染资源。

**Avoid frequent state switch**

通过批量处理同一集群内的网格数据，从而最小化CPU与GPU之间的状态切换开销，实现高效批量渲染。

## Nanite: State-of-the-Art in GPU-Driven Pipeline Architecture
