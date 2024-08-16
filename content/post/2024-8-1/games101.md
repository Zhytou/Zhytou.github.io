---
title: "GAMES101 学习笔记"
date: 2024-08-01T17:06:45+08:00
draft: false
---

- [Transformation](#transformation)
- [Rasterization](#rasterization)
  - [Anti-aliasing](#anti-aliasing)
  - [Visibility/Occlusion](#visibilityocclusion)
- [Shading](#shading)
  - [Illumination Model](#illumination-model)
  - [Lambert's Cosine Law](#lamberts-cosine-law)
  - [Phong/Blinn-Phong Model](#phongblinn-phong-model)
  - [Shading Frequency](#shading-frequency)
  - [Graphics Pipeline](#graphics-pipeline)
- [Texture](#texture)
  - [Texture Mapping](#texture-mapping)
  - [Interpolation Across Triangles: Barycentric Coordinates](#interpolation-across-triangles-barycentric-coordinates)
- [Shadow Mapping](#shadow-mapping)
  - [Shadow Mapping Process](#shadow-mapping-process)
  - [Drawbacks](#drawbacks)
- [PBR](#pbr)
  - [Radiometry](#radiometry)
  - [Microfacet Model](#microfacet-model)
  - [BRDF](#brdf)
  - [Rendering Equation](#rendering-equation)
  - [Cook-Torrance BRDF](#cook-torrance-brdf)
  - [PBR Implementation](#pbr-implementation)
  - [PBR GI——IBL](#pbr-giibl)
- [Ray Tracing](#ray-tracing)
- [References](#references)

## Transformation

- 模型变换（Model Transformation）：模型变换指的是对3D场景中的单个对象或模型进行变换的过程。这种变换包括对对象的顶点进行缩放、旋转和平移，将对象定位和定向到世界坐标系中。
- 视图/相机变换（View/Camera Transformation）：视图或相机变换涉及将整个场景从世界坐标系转换到相机或视图坐标系的过程。这种变换定义了虚拟相机在场景中的位置和方向。它包括设置相机位置、定义目标点或观察方向以及指定上方向等操作。
- 投影变换（Projection Transformation）：投影变换是将3D场景的坐标转换为屏幕上的2D坐标的过程。这种变换模拟了3D场景如何投影到2D平面上，考虑到透视和深度提示。常见的投影类型包括透视投影和正交投影。

其中，模型和视图变换通常都放在顶点着色器阶段，而投影变换则由图形流水线自动实现（裁剪/透视剔除）。

## Rasterization

光栅化（Rasterization）就是把几何图元转化成片元（像素）的过程。其中，光栅（Raster）一词来自德语，表示栅格。因此，光栅化的通俗理解就是在栅格显示设备上绘制图像。

**基本图元**：

一般来说，光栅化的输入都是三角形图元。而采用三角形作为基本图元的原因包括：

- 三角形是最基本的多边形。
- 三角形具有平面性。
- 三角形可以明确定义内部和外部，且可以通过向量叉积来判断。
- 任意多边形可以拆分成多个三角形。

**采样绘制**：

采样绘制就是将三角形图元分割为屏幕上的像素，确定哪些像素需要在稍后上色。具体来说，就是根据像素中心是否位于图元内部这一点来判断的，如下图所示。

![采样绘制](https://i-blog.csdnimg.cn/blog_migrate/0f3b07a11380d1754bbabc63d6a9e6ab.png)

**判断点在三角形内部**：

设三角形顶点为P1、P2和P3，待检测点为Q，则Q在三角形P1P2P3内部的条件为，下面三个叉乘符号相同：

- P0P1×P0Q
- P1P2×P1Q
- P2P0×P2Q

### Anti-aliasing

**走样原理**：

站在信号与系统的角度来看采样，在时域上是原信号和周期冲激信号的一个乘积。那么根据傅里叶变化，在频域上就是原信号和周期冲激信号的卷积，如下图所示。

![傅里叶变换](https://i-blog.csdnimg.cn/blog_migrate/e66dc5b5fe031c6065421bafc7d88b99.png)

那么当原信号频谱过宽或采样用的冲激信号频率太低，就容易发生信号重叠，由此造成走样。

![走样原理](https://i-blog.csdnimg.cn/blog_migrate/5a79462c87eee40450393d63c45bb780.png)

**抗锯齿/抗走样原理**：

根据走样原理，由此可得出抗锯齿原理其实就是尽可能避免信号高频部分发生重叠。一般来说，常见的抗锯齿方法有两种，分别是：

- **模糊+采样（Blur Anti-aliasing）**：即先使用一个平滑滤波器去滤除部分高频信号，再进行采样。
- **超采样抗锯齿（Supersampling Anti-aliasing，SSAA）**：即使用一个更高的采样频率来避免潜在的重叠。

**多重采样抗锯齿**：

此外，还需要补充的是多重采样抗锯齿(Multisample Anti-aliasing, MSAA)。它借鉴了超采样的思想，但同时又避免了超采样的性能消耗。因为在前面提到的SSAA中，每个子采样点都要进行单独的着色，这样在片元着色器比较复杂的情况下其实是非常浪费性能的。

相比之下，MSAA虽然增加了每个像素点的子采样点数量，但对于每个像素来说都只需要运行一次片元着色器。因为，它使用采样点的覆盖率来计算该像素的颜色。同样的，深度和模板测试也受多个采样点的影响。

### Visibility/Occlusion

**画家算法**：受画家作画启发，根据片元位置从后往前依次着色，计算framebuffer值。此时，后着色的片元颜色会覆盖旧的framebuffer值。对于首位交替重叠的特殊情况难以处理。

**Z-Buffer算法**：额外添加一个z-buffer记录当前像素的最小z值。

## Shading

> In computer graphics, shading is the process of applying a material to an object.

着色（Shading）是计算机图形学中用于模拟光照效果的过程，通过对物体表面的光照属性进行计算，为物体赋予逼真的外观。

### Illumination Model

光照模型（Illumination Model），有时也被称为着色模型，是用于计算任意给定位置的光线强度的数学模型。根据是否考虑光源和场景中物体相互作用产生的二次光照，可将光照模型分为局部光照模型和全局光照模型。其中，常见的局部光照模型包括：Lambert漫反射模型、Phong光照模型、Blinn-Phong光照模型和Cook-Torrance模型等。而全局光照模型则包括：光线追踪、路径追踪、辐射度算法和光子映射等。

具体的，局部光照大都基于经验（当然也有Cook-Torrance模型这种PBR的特例），而全局光照模型是基于光学物理原理的。后者的计算依赖于光能在现实世界中的传播情况，会考虑光线与整个场景中各物体表面及物体表面间的相互影响，包括多次反射、透射、散射等。正是这个原因，全局光照模型通常需要相当大的计算量，但同时也能取得非常逼真的真实效果。此外，局部光照模型往往无法直接生成阴影，而全局光照模型则可以。

### Lambert's Cosine Law

Lambertian模型用于描述粗糙材料的表面。它是一种理想模型，只考虑漫反射，即完全漫反射。这些反射光，在不同角度的辐射强度会依余弦公式变化，角度越大强度越弱。

![Lambert余弦定理](https://i-blog.csdnimg.cn/blog_migrate/de1476d341f40073b443de3aec2450da.png#pic_center)

### Phong/Blinn-Phong Model

在Phong/Blinn-Phong光照模型中，它将任意位置发出的光线分成漫反射(Diffuse/Lambertian)、镜面(Specular)和环境(Ambient)三个分量。

假设光源强度为I，带计算位置和光源距离为r，则该位置入射光线强度$L_{in}=I/r^2$。此时，出射光线强度为三个分量之和，即$L_{out}=L_d+L_s+L_a$。

**Diffuse/Lambertian**：

其中，漫反射强度由入射光线强度$L_{in}$、材料漫反射系数$K_d$、法向量方向$\vec{n}$和反射光线方向$\vec{out}$一起决定，即$L_d=k_d*L_{in}*max(0, \vec{n}*\vec{out})$。

**Specular**：

类似的，高光强度（或镜面反射强度）也由入射光线强度$L_{in}$、材料镜面反射系数$K_s$、材料的高光反光度shininess、法向量方向$\vec{n}$和反射光线方向$\vec{out}$一起决定。其中，高光反光度描述的是材料对镜面反光散射的能力。一个物体的反光度越高，反射光的能力越强，散射得越少，高光点就会越小。在下面的图片里，你会看到不同反光度的视觉效果影响：

![高光反光度](https://learnopengl-cn.github.io/img/02/02/basic_lighting_specular_shininess.png)

尽管Phong模型和Blinn-Phong模型计算这一分量所需参数类似，但计算方法却不同，这也是二者的区别。其中，前者的计算方式如下$L_s=k_s*L_{in}*pow(max(0, \vec{in}*\vec{out}), shininess)$。而后者则是引入了一个名为$\vec{h}$的半程向量来改进前者计算中出现的高光不连续问题，其示意图如下。具体来说，$\vec{h}=\frac{\vec{in}+\vec{out}}{||\vec{in}+\vec{out}||}$，而高光强度$L_s=k_s*L_{in}*pow(max(0, \vec{n}*\vec{h}), shininess)$。

![半程向量](https://zhytou.github.io/post/2024-8-1/half_vector.png)

**Ambient**：

至于环境光的计算方式就非常简单了。它表示为光的颜色乘以一个很小的常量环境因子，再乘以物体的颜色。

### Shading Frequency

- 面着色（Flat Shading）：一个平面计算一次，计算结果应用给该平面的每个点
- 顶点着色（Vertex Shading/Grouraud Shading）：每个顶点都计算一次，然后面内点做插值
- 像素着色（Pixel Shading/Phong Shading）：算出每个顶点法线，对法线进行插值得到面内每个像素的法线，再做着色计算

### Graphics Pipeline

- 应用阶段：输入一堆三维空间中的点。
- 顶点处理：经过MVPV变换后得到这些点在屏幕上的二维坐标。
- 图元处理：规定哪些点相互构成三角形(指定图元拓扑结构)。
- 光栅化：得到被三角形覆盖的像素。
- 片元处理：计算每个像素该显示什么，应该是什么颜色，Z-buffer的生成等。

## Texture

### Texture Mapping

### Interpolation Across Triangles: Barycentric Coordinates

## Shadow Mapping

前面提到使用局部光照模型进行着色并不会生成阴影，因此就需要使用额外的技术手段来生成这一信息。其中，最基础的方法就是阴影贴图（Shadow Mapping）技术。

### Shadow Mapping Process

### Drawbacks

## PBR

基于物理的渲染（Physicallly Based Rendering, PBR）是计算机图形学中一种主流的着色方法。相比于，传统的经验模型，如Phong式光照模型，PBR方法基于物理学原理和光学特性，更加真实和准确。

而其三大基础分别是辐射度量学、微平面模型和反射方程。其中，前两者的出现基本上只是为了证明：PBR里的数学公式是符合物理的；而最后的反射方程则是PBR中核心数学公式。

### Radiometry

辐射度量学（Radiometry）是究各种电磁辐射强弱的学科。它定义了一系列物理变量，用来从不同角度描述电磁辐射强度，包括：

- 辐射能（Radiant Energy）指电磁波中电场能量和磁场能量的总和，也叫电磁波的能量，常用符号Q表示。
- 辐射通量/功率（Radiant Flux/Power）指单位时间内产生的、反射的、接收的能量，常用符号$\Phi$表示。
- 辐射强度（Radiant Intensity）指单位立体角上，产生的、反射的、接收的辐射通量，常用符号I表示。
- 辐照度（Irradiance）指单位投影面积上接收到的辐射通量，常用符号E表示。
- 辐射率（Radiance）指单位投影面积收到/发出的单位立体角上的辐射通量，常用符号L表示。

其中，和计算机图形学相关的两个物理量其实是辐照度（Irradiance）和辐射率（Radiance）。它们的关系如下：

![Irradiance和Radiance的关系](https://pic3.zhimg.com/80/v2-7cddd5ba8c0e546af7245a60982cfcaa_1440w.webp)

### Microfacet Model

微表面模型是计算机图形学中用于模拟真实表面的微观结构的一种技术。它将现实世界中任意材料视作多个微小凹凸结构组成，并使用统计学原理去描述微观结构的几何形状、法线分布和表面粗糙度等参数从而计算光线的反射和折射。

### BRDF

BRDF(Bidirectional Reflectance Distribution Function)，译作双向反射分布函数，是一个用来描述物体表面如何反射光线的方程。简单来说，它就是微表面模型的数学表达。

具体的，BRDF表示了当给定一条入射光的时候，某一条特定的出射光线的性质是怎么样的。它的精确定义是出射光辐射率(Radiance)的微分和入射光辐照度(Irradiance)的微分之比，即$f(l,v)=\frac{dL_o(v)}{dE(l)}$。其中，v和l分别为出射和入射光线方向。

### Rendering Equation

渲染方程是一个描述光能在场景中流转的方程，它基于能量守恒定律，在理论上给出了一个完美的光能求解结果。在一个特定的位置和方向，出射光Lo是发射光Le与反射光之和。而反射光本身则是各个方向的入射光Li之和乘以表面反射率及入射角。

![渲染方程](https://pic2.zhimg.com/80/v2-6494cee4b32f7f654c724b2efb728285_1440w.webp)

其中，积分中的$f_r(x, \vec{w'}, \vec{w})$正是BRDF，即出射光辐射率的微分和入射光辐照度的微分之比。

### Cook-Torrance BRDF

> [learnopengl中关于Cook-Torrance BRDF的介绍](https://learnopengl-cn.github.io/07%20PBR/01%20Theory/#brdf)

### PBR Implementation

> [learnopengl中关于使用opengl实现一个PBR着色器](https://learnopengl-cn.github.io/07%20PBR/02%20Lighting/)

### PBR GI——IBL

> [learnopengl中漫反射IBL](https://learnopengl-cn.github.io/07%20PBR/03%20IBL/01%20Diffuse%20irradiance/)
> [learnopengl中镜面反射IBL](https://learnopengl-cn.github.io/07%20PBR/03%20IBL/02%20Specular%20IBL/)

## Ray Tracing

## References

- [GAMES101: 现代计算机图形学入门](https://sites.cs.ucsb.edu/~lingqi/teaching/games101.html)
- [CSDN GAMES101学习笔记](https://blog.csdn.net/motarookie/category_11510961.html)
- [LearnOpenGL](https://learnopengl-cn.github.io/)
- [知乎 图形学基础](https://zhuanlan.zhihu.com/p/430541328)
