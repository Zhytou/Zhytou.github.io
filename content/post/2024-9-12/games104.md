---
title: "Games104"
date: 2024-09-12T13:55:22+08:00
draft: false
---

- [1 Overview of Game Engine](#1-overview-of-game-engine)
- [2 Layered Architecture of Game Engine](#2-layered-architecture-of-game-engine)
- [3 How to Build a Game World](#3-how-to-build-a-game-world)
- [4 Rendering on Game Engine](#4-rendering-on-game-engine)
  - [Outline of Rendering](#outline-of-rendering)
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
  - [Classic Shadow Solution](#classic-shadow-solution)
  - [Moving Wave of High Quality](#moving-wave-of-high-quality)
  - [Shader Management](#shader-management)
  - [5 Summary](#5-summary)
- [6(1) Terrain](#61-terrain)
  - [Hard Tessellation](#hard-tessellation)
- [6(2) Sky and Atmosphere](#62-sky-and-atmosphere)
  - [Atmosphere](#atmosphere)
  - [Cloud](#cloud)
- [7 Render Pipeline, Post-process and Everything](#7-render-pipeline-post-process-and-everything)
  - [Ambient Occlusion](#ambient-occlusion)
  - [Fog](#fog)
  - [Anti-aliasing](#anti-aliasing)
  - [Post-processs](#post-processs)
  - [Render Pipeline](#render-pipeline)
- [8 Basics of Animation System](#8-basics-of-animation-system)
  - [Challenges in Game Animation](#challenges-in-game-animation)
  - [Outline of Animation System](#outline-of-animation-system)
  - [2D Animation Techniques in Games](#2d-animation-techniques-in-games)
  - [3D Animation Techniques in Games](#3d-animation-techniques-in-games)
  - [Skinned Animation Implementation](#skinned-animation-implementation)
- [9 Advanced Animation Technology](#9-advanced-animation-technology)
- [10 Physic System](#10-physic-system)
- [11 Tool Chains](#11-tool-chains)
  - [Outline of Tool Chains](#outline-of-tool-chains)
  - [What is Game Engine Tool Chain](#what-is-game-engine-tool-chain)
  - [Complicated Tool GUI](#complicated-tool-gui)
  - [How to Load Asset - Deserialization](#how-to-load-asset---deserialization)
  - [How to Make a Robust Tool](#how-to-make-a-robust-tool)
  - [How to Make Tool Chain](#how-to-make-tool-chain)
  - [What You See is What You Get](#what-you-see-is-what-you-get)
- [Advanced Topics of Tool Chain](#advanced-topics-of-tool-chain)
  - [Architecture of A World Editor](#architecture-of-a-world-editor)
  - [Plugin Architecture](#plugin-architecture)
  - [Design Narrative Tools](#design-narrative-tools)
  - [Reflection and Gameplay](#reflection-and-gameplay)
  - [Collaborative Editing](#collaborative-editing)

## 1 Overview of Game Engine

**Basic Elements**:

- engine structure and layer
- data organization and management

**Rendering**:

- model, material, shader and texture
- light and shadow
- render pipeline
- sky, terrain, etc

**Animation**:

- basic concepts of animation
- animation structure and pipeline

**Physics**:

- basic concepts of physics system
- gameplay applications
- performance optimization

**Gameplay**:

- event system
- scripts system
- graph driven

**Misc. System**:

- effects
- navigation
- camera

**Tool Set**:

- C++ reflection
- data schema

**Onlien Gaming**:

- lockstep synchronization
- state synchronization
- consistency

## 2 Layered Architecture of Game Engine

## 3 How to Build a Game World

## 4 Rendering on Game Engine

### Outline of Rendering

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

$$
L_o(x, w_o)=L_e(x, w_o)+\int_{H^2}L_i(x, w_i)f_r(x, w_i, w_o)cos\theta dw_i
$$

**Challenge**:

- Visibility to lights
- Light soure complexity
- How to integral efficiently on hardware
- Indirect light/global illumination

### Starting from Simple

**Simple Light Solution**:

- Use simple light soure(directional/point light) in most cases.
- Use ambient light to hack others.
- Use evironment map to enhance glossory surface reflection.
- Use evironment mipmap to represent different roughness of surface.

**Blinn-phong Material**：

在Blinn-Phong光照模型中，它将任意位置发出的光线分成漫反射(Diffuse/Lambertian)、镜面(Specular)和环境(Ambient)三个分量。

其中，漫反射强度由入射光线强度$L_{in}$、材料漫反射系数$K_d$、法向量方向$\vec{n}$和反射光线方向$\vec{out}$一起决定，即$L_d=k_d*L_{in}*max(0, \vec{n}*\vec{out})$。

而高光强度则由入射光线强度$L_{in}$、材料镜面反射系数$K_s$、材料的高光反光度shininess、法向量方向$\vec{n}$和半程向量$\vec{h}$一起决定。具体来说，$\vec{h}=\frac{\vec{in}+\vec{out}}{||\vec{in}+\vec{out}||}$，而高光强度$L_s=k_s*L_{in}*pow(max(0, \vec{n}*\vec{h}), shininess)$。

至于环境光的计算方式就非常简单了。它表示为光的颜色乘以一个很小的常量环境因子，再乘以物体的颜色。

尽管Blinn-Phong模型简单易实现，但其也有明显的问题，包括：

- 能量不守恒，出射能量可能大于入射能量
- 难以渲染复杂模型，大多有一种塑料感

**Shadow**:

目前，游戏引擎主流的阴影算法是一种名为阴影贴图(Shadow Map)的算法。它的核心思想是——对于指定光源来说，场景中某个点是否被其照亮，取决于从光源的视角看去，这个点是否可见。

具体来说，该算法被分成了两步：

- 第一步，从光源的视角出发绘制整个场景（平行光用正交投影，点光用透视投影），生成深度图，即所谓的阴影贴图；
- 第二步，从摄像机视角出发重新绘制场景，并根据光源投影矩阵的逆矩阵，将世界坐标空间变换回光源的投影空间，找出对应投影空间UV坐标以及投影空间内的深度d。

此时，就可以使用光源投影空间的UV坐标和阴影贴图，得到深度z。比较深度z和深度d，若d>z，则当前位置被遮挡，处于阴影内；反之，则未被遮挡。

阴影贴图也有一系列问题，包括：

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

**Basic Idea of IBL**: An image representing distant lighting from all directions.

**How to shade a point under IBL?**

Solving the rendering equation with Monte Carlo integration is a possible solution, but it's too slow.

A more elegant way is to use irradince map for diffuse light and split approximation for specular light.

### Classic Shadow Solution

**Big World and Cascade Shadow**:

- Partition the frustum into multiple frustums.
- A shadow map is rendered for each sub frustum.

**Blend between Cascade Layers**:

- A visible seam can be seen where cascades overlap between cascade layers because the resolution does not match.
- The shader then linearly interpolates between the two values based on the pixel's locationintheblendband

**PCF, PCSS and VSSM**:

[Games202 Shadow](https://zhytou.github.io/post/2024-8-5/games202/#3-real-time-shadows)

### Moving Wave of High Quality

**Quick Evolving of GPU**:

- More flexible new shader model: compute shader/mesh shader/ray-tracing shader
- High performance parallel architecture: warp or wave architecture
- Fully opened graphics API: DirectX 12/Vulkan(glNext)

**Real-Time Ray-Tracing on GPU**:

**Real-Time Global Illumination**:

- Screen-space GI
- SDF based GI
- Voxel based GI
- RSM/RTX GI

**More Complex Material Model**:

- BSDF(Strand-based hair)
- BSSDF

### Shader Management

**Uber Shader and Variants**:

> Uber shader is a combination of shader for all possible light types, render passes and material type.

超着色器，也叫全能着色器，通过宏区别执行分支，进而生成各种渲染参数下的着色器，最终达到简化着色器管理的目的。

至于为什么使用宏，而是不if-else这类条件语句，则是因为GPU是基于SIMD的，它应该尽量避免分支带来的执行时间不一致，从而影响执行效率。

**Cross Platform Shader Compile**:

### 5 Summary

目前游戏引擎中流行的渲染方案

- lightmap/light probe
- PBR/IBL
- cascade shadow/VSSM

## 6(1) Terrain

**Heightfield**:

**Adaptive Mesh Tessellation**:

地形自适应

**Binary Triangle Subdivision**:

**Subdivision and T-Junction**:

**QuadTree-Based Subdivision**:

### Hard Tessellation

- DerictX 11: hull shader+tesslator+domain shader
- DerictX 12: mesh shader

## 6(2) Sky and Atmosphere

### Atmosphere

**Analytic Atmosphere Appearance Modeling**:

- pros:
- cons:

**Participating Meida**:

### Cloud

**Cloud Type**:

**Mesh-Based Cloud Modeling**:

- pros: high quality
- cons: overall expensive/don't support dynamic weather

**Billboard Cloud**:

- pros: efficient
- cons: limited visual effect/limited cloud type

**Volumentric Cloud Modeling**:

- pros:
- cons: efficiency must be considered

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

屏幕空间环境光遮蔽(Screen Space Ambient Occlusion, SSAO)是对预计算AO贴图的一种改进。它只利用屏幕空间的信息就可以实时的计算出上述提到的visibility积分，即根据z-buffer信息得出着色点之间的相互遮挡情况。具体来说，SSAO在当前着色点周围采样一定数量的点，并计算出这些采样点的深度。接着，SSAO将其和阴影贴图中相应位置的深度比较，得出是否能接受到直接光。换句话说，只要采样点能接收到直接光，就认为它发出的间接光一定会被着色点接收到。

显然，这样大胆的假设会带来不少问题，包括：

- 只采样有限范围的点，即较远位置发出的间接光对着色点没有考虑；
- 理论上应该计算的是着色点到采样点的visibility，但在实践中只考虑的采样点到光源的visibility。
- 采样球，能量会不守恒。
- 由于没有法线信息，无法考虑cos。

**Horizon Based Ambient Occlusion**:

屏幕空间水平基准环境光遮蔽(Horizon Based Ambient Occlusion, HBAO)是对SSAO的一种改进。它整体和SSAO思路类似。只不过HBAO要求获取着色点法线信息，因此它可以采样半球同时考虑cos。此外，HBAO还引入的了光线步进(Ray Marching)来计算采样点到着色点之间的可见性，从而大大增强了其真实性。

**Ground Truth Based Ambient Occlusion**:

基于地面实况的环境光遮蔽(Ground Truth Based Ambient Occlusion, GTAO)基于HBAO，不过在AO积分公式中引入了一个cos项，同时对两个方向进行光线步进。

[GTAO](https://developer.huawei.com/consumer/cn/forum/topic/0202342980490920385)

**Ray-Tracing Ambient Occlusion**:

由于现代GPU已经提供了光线追踪的能力，所以使用在屏幕空间使用光线追踪，即屏幕空间反射(Screen Spaces Reflection, SSR)来实现全局光照变得越来越流行。相比其他AO算法，SSR可以感知到距离较远的物体发射的间接光，且可以不再假设间接光一定为diffuse的。

### Fog

**Depth Fog**:

**Height Fog**:

**Voxel-based Volumetric Fog**:

### Anti-aliasing

**Reason of Aliasing**:

Aliasing is a series of rendering artifact which is caused by high-frequency signal vs. insufficient of limited rendering resolutions.

本质上是需要用有限的屏幕像素去拟合连续的真实世界物体。

**Anti-aliasing**:

The general strategy of screen-based anti-aliasing schemes is using a sampling pattern to get more samples and then weight and sum samples to produce a pixel color.

[Games101 Anti-aliasing](https://zhytou.github.io/post/2024-8-1/games101/#aliasing)

### Post-processs

Post-process in 3D Graphics refers to any algorithm that will be applied to thefinal image. It can be done for stylistic reasons (color correction, contrast, etc.) or for realistic reasons (tone mapping, depth of field, etc.)

**Bloom**:

泛光(Bloom)是一种使得亮光部分产生“泛光”或“辉光”效果的技术。它用于模拟真实相机成像的瑕疵效果，即产生从图像明亮区域边界向外延伸的光线条纹，从而给人的感觉是极其明亮的光线压制住了摄像机或是透过眼睛看到该场景。

泛光一般的做法如下：

- 检测高光部分，使用rgb值构成的多项式，比如：r×0.2126+g×0.7152+b×0.0722。
- 对提取的亮部分进行高斯模糊处理，产生一个光晕效果。
- 多次迭代高斯模糊来实现不同程度的“泛光”效果。（高斯金字塔）
- 将模糊处理后的亮部分与原始图像进行混合合成，最终得到泛光效果。

**Tone Mapping**:

> The purpose of the Tone Mapping function is to map the wide range of highdynamicrange (HDR) colors into standard dynamic range (SDR) that a display can output.

颜色映射(Tone Mapping)是一种用于将HDR图像转换为LDR图像的技术。在HDR图像中，像素的亮度值可以超出典型的显示设备所能表示的范围，因此需要进行色调映射以适应标准的显示器或打印设备。

在实践中，颜色映射主要使用一个多项式，即色调映射曲线，来进行变化。

**Color Grading**:

色彩分级(Color Grading)是一种用于调整图像色彩和色调的技术，常用于电影制作和摄影中。通过对色彩、对比度、亮度等参数的调整，可以改变图像的整体感觉和氛围，达到艺术和表现效果。

### Render Pipeline

**Forward Rendering**:

前向渲染(Forward Rendering)是渲染基础方法。它会遍历每一个图元和光源，依次计算该图元涉及的像素颜色。其伪代码如下：

```txt
for(auto mesh : meshes) {
  for(auto light : lights) {
    color = shading(mesh, light);
  }
}
```

**Sort and Render Transparent after Opaque Objects**:

对于一个包含透明物的场景，前向渲染总是优先渲染不透明物体，接着将透明物体排序由远及近的渲染。至于，下图出现的难以排序的情况，则直接不管了。

![draw transparent](https://zhytou.github.io/post/2024-9-12/transparent.png)

**Deferred Rendering**:

延迟渲染(Deferred Rendering)是目前较为流行的一种渲染方法。它先遍历场景中图元，并将图元的几何信息存储在缓冲中，然后在后处理阶段根据光照信息计算最终颜色。其伪代码实现如下：

```txt
for(auto mesh : meshes) {
  write(g_buffers, mesh);
}

for (auto pixel : pixels) {
  g_buffer = read(g_buffers);
  for (auto light : lights) {
    shading(g_buffer, light);
  }
}
```

相比前向渲染，延迟渲染大大减少了shading函数调用次数，使其只需要针对可被照亮的片元计算。不过，它大量使用G-buffer，导致其内存和带宽开销非常大。此外，它不支持透明物体绘制且对MSAA不友好。

**Tile-based Rendering**:

基于块的渲染是一种在移动端广泛应用的渲染技术。它将屏幕分成小块（每块包含一定数量的像素），每个块独立处理，以提高渲染效率。

```txt
```

## 8 Basics of Animation System

### Challenges in Game Animation

**Interactive and dynamic animation**:

- Vary according to the interaction
- Cooperate with other gameplay systems
- Make adjustments in complex environments

**Real-time**:

- Compute per frame
- Massive animation data

**Realism**:

- More vivid expression
- More authentic experience

### Outline of Animation System

Basics of Animation Technology

- 2D Animation
- 3D Animation
- Skinned Animation Implementation（蒙皮动画）
- Animation Compression
- Animation DCC

Advanced Animation Technology

- Animation Blend
- Inverse Kinematics
- Animation Pipeline
- Animation Graph
- Facial Animation
- Retargeting

### 2D Animation Techniques in Games

**Sprite Animation**:

**Live2D**:

### 3D Animation Techniques in Games

**Degrees of Freedom**:

Degrees of freedom(DoF) refer to the number of independent variables or parameters of a system.

**Rigid Hierachical Animation**:

**Pre-vertex Animation**:

### Skinned Animation Implementation

**How to Animate a Mesh**:

## 9 Advanced Animation Technology

## 10 Physic System

## 11 Tool Chains

### Outline of Tool Chains

**Foundation of Tool Chains**:

- What is Game Engine Tool Chains
- Complicated Tool GUI
- How to Load Assets - Deserialization
- How to Make a Robust Tool
- How to Make Tool Chain
- What You See is What You Get
- One More Thing - Plugin

**Application & Advanced Topic**:

- Common Game Production Workflow
- Common Editors
- Reflection
- Collaborative Editing

### What is Game Engine Tool Chain

![Tool Chain Architecture](tool_chain)

引擎中的工具链其实是用户和Engine Runtine之间的中间层。它提供上图绿色线框中所提供功能，包括：蓝图编辑器、第三方资产导入、脚本管理等，其核心目的是允许让developers、artists以及designers能够一起工作。

此外，在商业引擎中Tool Chain往往比Engine Runtime复杂度更高，也更难以开发和维护。

### Complicated Tool GUI

**Immediate Mode**:

GUI通常有两种实现方式，即：及时模式(Immediate Mode)和保留模式(Retained Mode)。其中，前者是一种简单直接的GUI实现方式，它总是立刻响应用户，并重新绘制整个界面；而后者基于对象模型来描述和管理GUI，它会保存界面状态，每次仅更新发生变化的部分。

在及时模式下，GUI每一帧都需要重新绘制整个界面，响应用户输入和更新界面状态。换句话说，用户界面的状态和视图是即时生成的，不需要保留或存储界面的状态。它的特点以及优缺点如下：

- Characteristic: lightweight/procedural programming/widgets don't maintain any data or state
- Pros: straightforward/simple/quick prototype
- Cons: poor scalability/poor performance/poor maintainability

**Retained Mode**:

在保留模式下，界面元素和状态会被保留在内存中，每次只需要更新发生变化的部分，而不需要重新绘制整个界面。它的特点以及优缺点如下：

- Characteristic: object-oriented/widgets contain their own state and data/draw widgets as needed
- Pros: high scalability/high performance/high maintainability
- Cons: complex for developers(callbacks/synchronization between GUI)
- Examples: Unreal UMG/WPF GUI/QT

**Design Pattern**:

- MVC
- MVP
- MVVM

**Serialization and Deserialization**:

序列化（Serialization）是指将对象转换为字节流或其他格式，以便在网络传输或持久化存储时进行传输或保存的过程。解序列化（Deserialization）则是将序列化后的数据重新转换为对象的过程。

常见的序列化存储格式包括：

- Text files(can be read/good for debug): txt/json/yaml/xml
- Binary files(need less storage): uasset/fbx binary

**Asset Reference**:

Asset reference is a way to seperate redundant data into asset files and complete association by establishing reference relationshops.

**Build Variance by Data Inheritance**:

Inherit the data of the inherited object and allow overriding assignments to the data defined in its data structure.

### How to Load Asset - Deserialization

### How to Make a Robust Tool

**Core Functions**:

- Undo and Redo
- Crash Recovery

**How to Implement Undo/Redo/Crash Recovery?**:

Abstract all user operations to atomic commands which can invoke, revoke and serialize, deserialize.(Similar to crash recovery of DB)

**How to Define a Command**:

```java
public interface ICommand<TData>
{
  long UID;
  TData Data;

  void Invoke();
  void Revoke();
  byte[] Serialize();
  void Deserialize(byte[] data);
}
```

- UID: monotonic increase over time/unique identification
- Serialize/Deserialize: usually provide by GO data

### How to Make Tool Chain

**Challenge**: various tools for different users

- different views for different tools
- each tool has its own data structure
- same data may have different view for different user

develop all tools seperately?

- no saclability
- no maintainbility

**Schema**: a description structure

two definition ways:

- defined in code(oop)
- standalone schema definition(xml/json use reflection)

### What You See is What You Get

- Stanealone Tool
- In Game Tool: Editor Mode/Play in Editor

## Advanced Topics of Tool Chain

### Architecture of A World Editor

World editor is a hub for everything to build the world.

### Plugin Architecture

### Design Narrative Tools

### Reflection and Gameplay

**Reflection**:

In computer science, reflective programming or reflection is the ability of a process to examine, introspect, and modify its own structure and behavior.

By generating a code meta information map, reflection build the bridge between code and tools

**How to Implement Reflection in C++**:

- Collect type info from code
- Generate code to provide accessors for fields and methods
- Manage all accessors with a <string,accessor> map

**Code Rendering**:

### Collaborative Editing
