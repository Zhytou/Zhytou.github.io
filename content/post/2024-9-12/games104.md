---
title: "Games104"
date: 2024-09-12T13:55:22+08:00
draft: false
---

- [4 Rendering on Game Engine](#4-rendering-on-game-engine)
  - [Outline](#outline)
  - [GPU](#gpu)
  - [Renderable](#renderable)
  - [Render Objects in Engine](#render-objects-in-engine)
  - [Visibility Culling](#visibility-culling)
  - [Texture Comperssion](#texture-comperssion)
  - [Authoring Tools of Modeling](#authoring-tools-of-modeling)
  - [Cluster-based Mesh Pipeline](#cluster-based-mesh-pipeline)
  - [4 Summary](#4-summary)
- [5 Lighting, Materials and Shders](#5-lighting-materials-and-shders)
  - [Participants of Rendering Computation](#participants-of-rendering-computation)
  - [The Rendering Equation](#the-rendering-equation)
  - [Starting from Simple](#starting-from-simple)
  - [Pre-computed Global Illumination](#pre-computed-global-illumination)
  - [Physically Based Material](#physically-based-material)
  - [Image Based Lighting](#image-based-lighting)
  - [Shadow](#shadow)
  - [Moving Wave of High Quality](#moving-wave-of-high-quality)
  - [Shader Management](#shader-management)
  - [5 Summary](#5-summary)
- [6 游戏中地形大气和云的渲染](#6-游戏中地形大气和云的渲染)
  - [Hard Tessellation](#hard-tessellation)
- [7 Render Pipeline, Post-process and Everything](#7-render-pipeline-post-process-and-everything)
  - [Ambient Occlusion](#ambient-occlusion)
  - [Fog](#fog)
  - [Anti-aliasing](#anti-aliasing)
  - [Post-processs](#post-processs)

## 4 Rendering on Game Engine

### Outline

**Basics of Game Rendering**:

- Hardware architecture
- Render data organization
- Visibility

**Special Rendering**:

- Terrain
- Sky/Fog
- Postprocess

**Materials, Shaders and Lighting**:

- PBR(SG, MR)
- Shader permutation
- Lighting
  - Point/Directional lighting
  - IBL/Simple GI

**Pipeline**:

- Forward, deferred rendering, foward plus
- Real pipeline with mixed effects
- Ring buffer and V-Sync
- Time-based rendering

### GPU

**SIMD and SIMT**:

- Single Instruction Multiple Data
- Single Instruction Multiple Threads

**GPU Architecture**:

- GPC(Graphics Processing Cluster): A dedicated hardware block for
computing, rasterization, shading, and texturing, usually consisting of serval TPCs.
- TPC(Texture Processing Cluster): A sub-block mainly used to process texture data(texture sampling and filtering), consisting of 2 SM.
- SM(Streaming Multiprocesseor): Part of the GPU that runs CUDA kernels, containing multiple CUDA cores.
- CUDA Core: A computing unit in GPU.
- Warp: A collection of threads.

**CPU and GPU Dataflow**:

![CPU and GPU Dataflow](https://zhytou.github.io/post/2024-9-12/dataflow_cpu2gpu.png)

**GPU Bounds and Performance**:

- Memory Bounds
- ALU Bounds
- TMU(Texture Mapping Unit) Bounds
- BW(Bandwidth) Bounds

### Renderable

**Mesh Render Component**:

游戏世界中的任意物体都是GO，也即Game Object。而GO总是使用基于组件(Componet-based)的方式描述，它使得游戏功能可被分解为独立模块，易于维护和开发。

其中，一个可显示的GO往往拥有一个Mesh Render Component来帮助其显示。

**Mesh Primitive**:

图元/片元通常是一个三角形，即由三个顶点数据构成，比如：

```c++
struct Vertex {
  Vector3 m_position;
  // other info
  Ubyte m_color;
  Vector3 m_normal;
};
struct Triangle {
  Vertex m_vertex[3];
};
```

**Vertex and Index Buffer**:

实际上，图元只存储顶点序号，这样可以避免重复存储相同顶点（因为图元可能会共用多个顶点）。比如，OpenGL中用EBO存储序号、VAO存储顶点数据类型（说明每个Vertex中有哪些信息，怎么排布的）、VBO存储实际顶点数据。

**Why We Need Pre-vertex Normal**:

为什么每个顶点中都要保存一个单独法线信息？因为每个片元在光栅化后都会根据其图元顶点的法线插值得到一个新的法线，这样就使得模型拥有了更丰富的表达。

**Material**:

在游戏中，材质通常只描述物体的视觉特性，即物体如何和光交互。它不会包含物体的其他物理特性，比如密度、质量等。

**Famous Material Model**:

- Phong Model
- Physically Based Material
- Subsurface Material

### Render Objects in Engine

**Coordinate System and Transformation**:

- 模型变换（Model Transformation）：模型变换指的是对3D场景中的单个对象或模型进行变换的过程。这种变换包括对对象的顶点进行缩放、旋转和平移，将对象定位和定向到世界坐标系中。
- 视图/相机变换（View/Camera Transformation）：视图或相机变换涉及将整个场景从世界坐标系转换到相机或视图坐标系的过程。这种变换定义了虚拟相机在场景中的位置和方向。它包括设置相机位置、定义目标点或观察方向以及指定上方向等操作。
- 投影变换（Projection Transformation）：投影变换将场景从相机坐标转换到裁剪空间坐标，再进一步将其转换到NDC空间。
- 视口变换(Viewport Transformation)：将处于标准平面映射到屏幕分辨率范围之内，即[-1,1]^2[0,width]*[0,height], 其中width和height指屏幕分辨率大小。

**How to Display Different Textures on a Single Model? SubMesh/Resource Pool**:

将每一个可绘制的GO划分为SubMesh，每个SubMesh使用不同的纹理进行渲染。

### Visibility Culling

**Potential Visibility Set**:

**GPU Culling**:

### Texture Comperssion

纹理压缩(Texture Compression)是一种专为在三维计算机图形渲染系统中存储纹理而使用的图像压缩技术。与普通图像压缩算法的不同之处在于，纹理压缩算法为纹素的随机存取做了优化。

**In-game Texture Comperssion Requirement**:

- Decoding speed
- Random access(That's why cannot use JPEG)
- Compreesion rate and visual quality
- Encoding speed

**Block Compression**:

现代纹理压缩算法的基础是块压缩(Block Compression)。它将纹理数据分成4×4大小的块来进行颜色压缩，在渲染前将其解压。

目前比较流行的块压缩格式包括：

- On PC, BC7 or DXTC
- On mobile, ASTC or ETC

### Authoring Tools of Modeling

- PolyModeling: 3ds Max/Maya/Blender
- Sculpting: ZBrush
- Scanning
- Procedural Modeling: Houdini

### Cluster-based Mesh Pipeline

GPU-driven Rendering Pipeline(2015): Mesh Cluster Rendering

- Arbitray number of meshes in single drawcall.
- GPU-culled by cluster bounds.
- Cluster depth sorting.

Geometry Rendering Pipeline Architecture(2021): Rendering primitives are divided as:

- Batch
- Surf
- Cluster

**Programmable Mesh Pipeline**:

**Nanite**:

### 4 Summary

- The design of game engine is deeply related to the hardware architecture design.
- A submesh design is used to support with multiple materials.
- Using culling algorithms to draw as few objects as possible.
- As GPU become more powerful, more and more work are moved into GPU, which called GPU Driven.

## 5 Lighting, Materials and Shders

### Participants of Rendering Computation

- Lighting: Photon emit, bounce and perception is the origin of everything in rendering.
- Material: How matter react to photon
- Shader: How to train and organize those micro-slaves to finish such a vast and dirty

### The Rendering Equation

$$L_o(x, w_o)=L_e(x, w_o)+\int_{H^2}L_i(x, wi)f_r()cos\thetadw_i $$

勉强

### Starting from Simple

**Blinn-phong**：

问题：

- 能量不守恒，出射能量可能大于入射能量
- 难以渲染复杂模型，大多有一种塑料感

**Shadow**:

目前，游戏引擎主流的阴影算法是一种名为阴影贴图(Shadow Map)的算法。

问题：

- 贴图分辨率有限，致使多个像素可能需要使用同一个深度值，进而引起走样/锯齿现象。从信号与系统上来说，就是因为两个信号频率不同，就可能致使互相遮挡的问题。
- 贴图深度进度受限，造成条纹状阴影。引入bias，又有可能引起模型浮空的问题。

**Basic Shading Solution**：

- Simple light
- Blinn-Phong material
- Shadow Map

### Pre-computed Global Illumination

**Problem of Ambient**:

只能同一变亮或者变暗

**How to Reprensent**:

****

**Light Probe and Reflection Probe**:

### Physically Based Material

**Mircofacet Theory**:

**BRDF Model Based on Microfact**:

$$
f_r(x, w_o, w_i)=\alpha\frac{abedo}{\pi}+\frac{DFG}{4}
$$

其中，$\alpha$是材料的粗糙度的。

**Normal Distribution Function**:

$$
NDF_{GGX}(\vec{n}, \vec{h}, \alpha)=\frac{\alpha^2}{\pi((\vec{n}*\vec{h})^2(\alpha^2-1)+1)^2}
$$

```GLSL
float D_GGX_TR(vec3 N, vec3 H, float a)
{
  float a2     = a*a;
  float NdotH  = max(dot(N, H), 0.0);
  float NdotH2 = NdotH*NdotH;

  float nom    = a2;
  float denom  = (NdotH2 * (a2 - 1.0) + 1.0);
  denom        = PI * denom * denom;

  return nom / denom;
}
```

**Geometric Attenuation Term(Self-Shadowing)**:

$$
G_{Smith}(\vec{l}, \vec{v})=G_{GGX}(\vec{l})G_{GGX}(\vec{v})\\
G_{GGX}(\vec{v})=\frac{\vec{n}*\vec{v}}{(\vec{n}*\vec{v})(1-k)+k}\\
k=\frac{(\alpha+1)^2}{8}
$$

**Fresnel Equation**:

$$
F_{Schlick}(\vec{h}, \vec{v}, F_0)=F_0+(1-F_0)(1-(\vec{v}*\vec{h}))^5
$$

**Disney Principled BRDF**:

**PBR Specular/Glossiness(SG)**:

**PBR Metallic/Roughness(MR)**:

### Image Based Lighting

**Basic Idea of IBL**:

### Shadow

**Big World and Cascade Shadow**:

将视锥体分成多片的

**Blend between Cascade Layers**:

**Pros and Cons of Cascade Shadow**:

pros:
方法。

- 的
**Percentage Closer Filter**:

**Percentage Closer Soft Shadow**:

**Variance Soft Shadow Map**:

### Moving Wave of High Quality

**Quick Evolving of GPU**:

- More flexible new shader model: compute shader/mesh shader/ray-tracing shader
- High performance parallel architecture: warp or wave architecture
- Fully opened graphics API: DirectX 12/Vulkan(glNext)

**Real-Time Ray-Tracing**:

**Real-Time Global Illumination**:

**More Complex Material Model**:

**Virtual Shadow Map**:

### Shader Management

**Uber Shader and Variants**:

> Uber shader is a combination of shader for all possible light types, render passes and material type.

超着色器，也叫全能着色器，通过宏区别执行分支，进而生成各种渲染参数下的着色器，最终达到简化着色器管理的目的。

至于为什么使用宏，而是不if-else这类条件语句，则是因为GPU是基于SIMD的，它应该尽量避免分支带来的执行时间不一致，从而影响执行效率。

**Cross Platform Shader Compile**:

### 5 Summary

目前游戏引擎中流行的渲染方案

- lightmap/lightprobe
- PBR/IBL
- cascade shadow/VSSM

## 6 游戏中地形大气和云的渲染

**Heightfield**:

**Adaptive Mesh Tessellation**:

地形自适应

**Binary Triangle Subdivision**:

**Subdivision and T-Junction**:

**QuadTree-Based Subdivision**:

### Hard Tessellation

- DerictX 11: hull shader+tesslator+domain shader
- DerictX 12: mesh shader

## 7 Render Pipeline, Post-process and Everything

### Ambient Occlusion

> Ambient occlusion is approximation of attenuation of ambient light due to occlusion.

环境光遮蔽(Ambient Occlusion, AO)是对场景中物体因相互遮蔽而引发的环境光衰减的一种近似技术。它和Cook-Torrence BRDF中几何遮蔽项G描述的是类似信息。只不过，几何遮蔽项G用于确保BRDF的能量守恒，并正确计算镜面发射光；而AO技术则用于实现全局光照，即环境光。具体来说，它会得出一个衰减值attenuation，并乘上间接光照，从而得出最终的片元颜色。

它的理论基础仍然是渲染方程，不过多了一个假设，即将环境光和其BRDF视作一个常量。根据这个假设，便可将出射光强度化简为下式：

$$
L_o(x, w_o)=\int_H^2L_i(x, w_i)f_r(x, w_o, w_i)V(x, wi)cos\theta dw_i\\
L_o(x, w_o)=L_if_r\int_H^2V(x, wi)cos\theta dw_i\\
$$

某种意义上，它和阴影贴图一起实现了各类环境下的阴影现象。前者可以近似出多次弹射的间接光产生的阴影，即间接光阴影；而后者则只考虑少量点光源组成场景的阴影效果，即直接光阴影。

**Precomputed AO**：

早期的AO技术以预计算为主。它使用光线追踪得到一张AO贴图，即原理中提到的关于visibility的积分，从而在实时渲染中实现环境光遮蔽效果。不过这种方法的缺点也比较明显，它需要额外的纹理存储，且只能表现静态场景。

**Screen Space Ambient Occlusion**:

屏幕空间环境光遮蔽(Screen Space Ambient Occlusion, SSAO)是对预计算AO贴图的一种改进。它只利用屏幕空间的信息就可以实时的计算出上述提到的visibility积分。其核心是把实时计算中得到的z-buffer信息近似看作几何场景的，因为z-buffer表示。

不过这个原理其实有点问题，即

SSAO+

**Horizon Based Ambient Occlusion**:

**Ground Truth Based Ambient Occlusion**:

**Ray-Tracing Ambient Occlusion**:

由于现代GPU已经提供了

### Fog

**Depth Fog**:

**Height Fog**:

### Anti-aliasing

**Reason of Aliasing**:

Aliasing is a series of rendering artifact which is caused by high-frequency signal vs. insufficient of limited rendering resolutions.

本质上是需要用有限的屏幕像素去拟合连续的真实世界物体。

**Anti-aliasing**:

The general strategy of screen-based anti-aliasing schemes is using a sampling pattern to get more samples and then weight and sum samples to produce a pixel color.

关于抗走样算法这一部分的详细介绍，可以查看games101笔记中的介绍。

### Post-processs

**Bloom**:

**Tone Mapping**:

****:
