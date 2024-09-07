---
title: "GAMES101 学习笔记"
date: 2024-08-01T17:06:45+08:00
draft: false
---

- [Transformation](#transformation)
  - [MVP](#mvp)
  - [Triangle](#triangle)
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
  - [Light Mapping](#light-mapping)
  - [Normal Mapping](#normal-mapping)
  - [Shadow Mapping](#shadow-mapping)
- [PBR](#pbr)
  - [Radiometry](#radiometry)
  - [Optics and Microfacet Model](#optics-and-microfacet-model)
  - [BRDF and BxDF](#brdf-and-bxdf)
  - [Rendering Equation](#rendering-equation)
  - [Cook-Torrance BRDF](#cook-torrance-brdf)
  - [Diffuse Part](#diffuse-part)
  - [Specular Part](#specular-part)
  - [Linear Space and Tone Mapping](#linear-space-and-tone-mapping)
  - [PBR Implementation](#pbr-implementation)
  - [PBR GI——IBL](#pbr-giibl)
- [Ray Tracing](#ray-tracing)
- [References](#references)

## Transformation

### MVP

- 模型变换（Model Transformation）：模型变换指的是对3D场景中的单个对象或模型进行变换的过程。这种变换包括对对象的顶点进行缩放、旋转和平移，将对象定位和定向到世界坐标系中。
- 视图/相机变换（View/Camera Transformation）：视图或相机变换涉及将整个场景从世界坐标系转换到相机或视图坐标系的过程。这种变换定义了虚拟相机在场景中的位置和方向。它包括设置相机位置、定义目标点或观察方向以及指定上方向等操作。
- 投影变换（Projection Transformation）：投影变换是将3D场景的坐标转换为屏幕上的2D坐标的过程。这种变换模拟了3D场景如何投影到2D平面上，考虑到透视和深度提示。常见的投影类型包括透视投影和正交投影。

其中，模型和视图变换通常都放在顶点着色器阶段，而投影变换则由图形流水线自动实现（裁剪/透视剔除）。

### Triangle

****

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
- 光栅化：得到被三角形覆盖的像素。（除此之外，顶点着色器传给片元着色器的数据会通过插值得出。）
- 片元处理：计算每个像素该显示什么，应该是什么颜色，Z-buffer的生成等。

## Texture

### Texture Mapping

纹理贴图(Texture Mapping)简单的理解就是将一张二维图像，按照一定的映射关系，将每个像素贴合到物体表面的对应位置。纹理技术可以增加物体表面的细节。

**纹理环绕**：

**Mipmap**：

### Light Mapping

### Normal Mapping

除了可以保存物体的颜色信息，纹理还可以用于保存物体的法线信息，也就是法线贴图(Normal Mapping)。它的主要作用就是通过(U, V)

### Shadow Mapping

和路径追踪这类方法不同，光栅化不会自动生成阴影。其中，最基础的方法就是阴影贴图（Shadow Mapping）技术。

## PBR

> [PBR 白皮书](https://github.com/QianMo/PBR-White-Paper/tree/master)

基于物理的渲染（Physicallly Based Rendering, PBR）是计算机图形学中一种主流的着色方法。相比于，传统的经验模型，如Phong式光照模型，PBR方法基于物理学原理和光学特性，更加真实和准确。具体来说，PBR理论包括：

- 基于物理的材质（Material）；
- 基于物理的光照（Lighting）；
- 基于物理适配的摄像机（Camera）。

而该理论的三大基础则分别是辐射度量学、微平面模型和反射方程。其中，前两者的出现基本上只是为了证明：PBR里的数学公式是符合物理的；而后者才是PBR中的核心。换句话说说，反射方程的推导、简化和应用构成了PBR理论。

最后，基于物理的渲染相比传统方法有以下优势：

- 直观参数（最重要的一点）：PBR使用的参数（如粗糙度、金属度等）与实际物理属性紧密相关，这使得艺术家和开发者能够更直观地理解和调整材质参数以获得预期的视觉效果。
- 一致的材质表现：PBR模型提供了一种标准化的方法来表示材质属性，这意味着同样的材质设置在不同的光照条件下能保持一致的外观。这种一致性对于跨平台开发和维护大型项目尤其重要。
- 逼真的视觉效果：由于PBR基于真实的物理光照模型，它能够更好地模拟光与材质的交互，提供更逼真的阴影、高光和反射效果。这种逼真效果能够显著提升视觉细节和沉浸感。

### Radiometry

辐射度量学（Radiometry）是究各种电磁辐射强弱的学科。它定义了一系列物理变量，用来从不同角度描述电磁辐射强度，包括：

- 辐射能（Radiant Energy）指电磁波中电场能量和磁场能量的总和，也叫电磁波的能量，常用符号Q表示。
- 辐射通量/功率（Radiant Flux/Power）指单位时间内产生的、反射的、接收的能量，常用符号$\Phi$表示。
- 辐射强度（Radiant Intensity）指单位立体角上，产生的、反射的、接收的辐射通量，常用符号I表示。
- 辐照度（Irradiance）指单位投影面积上接收到的辐射通量，常用符号E表示。
- 辐射率（Radiance）指单位投影面积收到/发出的单位立体角上的辐射通量，常用符号L表示。

其中，和计算机图形学相关的两个物理量其实是辐照度（Irradiance）和辐射率（Radiance）。它们的关系如下：

![Irradiance和Radiance的关系](https://pic3.zhimg.com/80/v2-7cddd5ba8c0e546af7245a60982cfcaa_1440w.webp)

### Optics and Microfacet Model

**微平面模型**：

微表面模型是计算机图形学中用于模拟真实表面的微观结构的一种技术。它将现实世界中任意材料视作多个微小凹凸结构组成，并使用统计学原理去描述微观结构的几何形状、法线分布和表面粗糙度等参数从而计算光线的反射和折射。

为了更好的理解微平面模型，下面将介绍一些光学基础知识。

**折射和折射率**：

而光在传播到两种不同介质交界处时，原始光波和新的光波的相速度（Phase Velocity）的比率定义了介质的光学性质，就是折射率（Index Of Refraction，IOR），由字母n表示。

除了代表光的相速度的实部n之外，还用希腊字母κ（kappa）表示介质将光能转为为其他形式能量的吸收性。n和κ通常都随波长而变化，两者组合成复数n +iκ，称为复折射率（complex index of refraction）。

也就是说，折射率IOR是一个复数（complex number），其分为实部和虚部两部分：

- 折射率的实部（real part）度量了物质如何影响光速，即相对于光在真空中传播速度减慢的度量。
- 折射率的虚部（imaginary part）确定了光在传播时是否被吸收，转换成其他形式的能量，通常是热能。非吸收性介质虚部为零。

**光与平面交互**：

![漫反射](https://raw.githubusercontent.com/QianMo/PBR-White-Paper/master/content/part%202/media/db572e0923acd8d22e67a4e1875fb206.png)

在了解了折射和折射率之后，我们可以进一步总结得出光与平面交互方法，包括：

- 反射（Reflection）。光线在两种介质交界处的直接反射即镜面反射（Specular）。金属的镜面反射颜色为三通道的彩色，而非金属的镜面反射颜色为单通道的单色。
- 折射（Refraction）。 从表面折射入介质的光，会发生吸收和散射，而介质的整体外观由其散射和吸收特性的组合决定，其中：
  - 散射（Scattering）。 折射率的快速变化引起散射，光的方向会改变（分裂成多个方向），但是光的总量或光谱分布不会改变。散射最终被视作的类型与观察尺度有关：
    - 次表面散射（Subsurface Scattering）。观察像素小于散射距离，散射被视作次表面散射。
    - 漫反射（Diffuse）。观察像素大于散射距离，散射被视作漫反射。
    - 透射（Transmission）。入射光经过折射穿过物体后的出射现象。透射为次表面散射的特例。
  - 吸收（Absorption）。 具有复折射率的物质区域会引起吸收，具体原理是光波频率与该材质原子中的电子振动的频率相匹配。复折射率（complex number）的虚部（imaginary part）确定了光在传播时是否被吸收（转换成其他形式的能量）。发生吸收的介质的光量会随传播的距离而减小（如果吸收优先发生于某些波长，则可能也会改变光的颜色），而光的方向不会因为吸收而改变。任何颜色色调通常都是由吸收的波长相关性引起的。

因此，从宏观上看，光在介质表面扣除吸收部分，即剩下：（镜面）反射、漫反射和透射三部分。

**菲涅尔效应和方程**：

### BRDF and BxDF

BRDF(Bidirectional Reflectance Distribution Function)，译作双向反射分布函数，是一个用来描述物体表面如何反射光线的方程。简单来说，它就是微表面模型的数学表达。

具体的，BRDF表示了当给定一条入射光的时候，某一条特定的出射光线的性质是怎么样的。它的精确定义是出射光辐射率(Radiance)的微分和入射光辐照度(Irradiance)的微分之比，即$f(l,v)=\frac{dL_o(v)}{dE(l)}$。其中，v和l分别为出射和入射光线方向。

其实，采用ωi和ωo作为输入参数的Blinn-Phong光照模型也可以被当作是一个BRDF。不过由于Blinn-Phong模型并没有遵循能量守恒定律，因此它不被认为是基于物理的渲染。至于，目前广泛用于基于物理渲染和实时渲染的BRDF模型则是一种被称为Cook-Torrance的BRDF模型。我们将在后两节详细介绍它。

除了BRDF之外，图形学中还有一些其他的双向分布函数，比如BRDF、BTDF、BSDF、BSSRDF等，这些一起被称作BxDF。

![BxDF](https://raw.githubusercontent.com/QianMo/PBR-White-Paper/master/content/part%201/media/8c60f94b8b6f430fc5dcb41068770454.png)

在上述这些BxDF中，BRDF最为简单，也最为常用。因为游戏和电影中的大多数物体都是不透明的，用BRDF就完全足够。而BSDF、BTDF、BSSRDF往往更多用于半透明材质和次表面散射材质。

### Rendering Equation

渲染方程是一个描述光能在场景中流转的方程，它基于能量守恒定律，在理论上给出了一个完美的光能求解结果。在一个特定的位置和方向，出射光Lo是发射光Le与反射光之和。而反射光本身则是各个方向的入射光Li之和乘以表面反射率及入射角。

![渲染方程](https://pic2.zhimg.com/80/v2-6494cee4b32f7f654c724b2efb728285_1440w.webp)

其中，积分中的$f_r(x, \vec{w'}, \vec{w})$正是BRDF，即出射光辐射率的微分和入射光辐照度的微分之比。

### Cook-Torrance BRDF

> [learnopengl中关于Cook-Torrance BRDF的介绍](https://learnopengl-cn.github.io/07%20PBR/01%20Theory/#brdf)

Cook-Torrance BRDF是一种目前最流行一种BRDF反射模型，它包含漫反射和镜面反射两个部分：$f_r=k_df_{lambert}+k_sf_{cook-torrance}$。其中，$f_{lambert}=\frac\rho\pi$，而$f_{cook-torrance}=\frac{DFG}{4(\vec{w_o}*\vec{n})(\vec{w_i}*\vec{n})}$。

### Diffuse Part

Cook-Torrance BRDF中，漫反射系数$f_{lambert}=\frac\rho\pi$的推导基于一个前提：入射光是均匀且遍布整个半球方向。那么出射光则可表示为：

$$
\begin{align}
L_o(w_o) &= \int_{H^2}{f_rL_i(w_i)cos\theta}dw_i\\
         &= f_rL_i\int_{H^2}{cos\theta}dw_i\\
         &= f_rL_i\pi\\
f_r &= \frac\rho\pi
\end{align}
$$

其中，$\rho$表示的是发生反射位置的颜色。

### Specular Part

Cook-Torrance BRDF的镜面反射部分包含法线分布函数(Normal Distribution Function)、菲涅尔方程(Fresnel Equation)、几何函数(Geometry Function)以及用于矫正的分母 。

其中，分母作为微观几何的局部空间和整个宏观表面的局部空间之间变换的微平面量的校正，有两点需要注意：

- 对于分母中的点积，仅仅避免负值是不够的 - 也必须避免零值。通常通过在常规的clamp或绝对值操作之后添加非常小的正值来完成。
- Microfacet Cook-Torrance BRDF是实践中使用最广泛的模型，实际上也是人们可以想到的最简单的微平面模型。它仅对几何光学系统中的单层微表面上的单个散射进行建模，没有考虑多次散射，分层材质，以及衍射。Microfacet模型，实际上还有很长的路要走。

**菲涅尔项**：

其中，菲涅尔方程描述的是被反射的光线对比光线被折射的部分所占的比率，这个比率会随着观察角度变化而变化。比如，当我们站在湖边低头看脚下的湖水，会发现水是透明的，反射不会特别强烈；而如果我们看远处的湖面时，会发现水并不透明，而且反射非常强烈。

目前，菲涅尔项一般都采用Schlick近似$$。因为计算成本低廉，而且精度足。

**法线分布函数**：

法线分布函数用于描述给定法线$\vec{n}$、半程向量$\vec{h}$和粗糙度$\alpha$时，法线方向和半程向量方向一致的比例。它满足一些列基本性质，包括：

- 微平面法线密度始终为非负值；
- 微表面的总面积始终不小于宏观表面总面积；
- 任何方向上微观表面投影面积始终与宏观表面投影面积相同；
- 若观察方向为法线方向，则其积分可以归一化。

一个最常见的法线分布函数其实就是Blinn-Phong中关于镜面反射引入的shiness参数，其具体表达如下：$NDF(\vec{n}, \vec{h}, shiness)=pow(max(0, \vec{n}*\vec{h}), shininess)$。当粗糙度越小（或反光度越大），镜面反射光线越集中，物体表面越有光泽；反之，镜面反射越散射，物体表面越缺乏光泽。

![法线分布函数](https://learnopengl-cn.github.io/img/07/01/ndf.png)

除此之外，还有比较经典的CGX法线分布函数$NDF(\vec{n}, \vec{h}, \alpha)=\frac{\alpha^2}{\pi((\vec{n}*\vec{h})^2(\alpha^2-1)+1)^2}$。将它在GLSL实现可得：

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

注意：NDF其实和传统的Normal Mapping有一定相似。只不过，法线贴图只为每个像素提供。而法线分布函数使用法线贴图和粗糙度贴图一起，提供了亚像素精度的细节。且传统法线贴图没有经过归一化处理，不满足能量守恒，容易出现失真。

**几何函数**：

几何函数用于描述给定法线$\vec{n}$、视线$\vec{v}$和粗糙度$\alpha$时，反射光线被遮蔽的比率，它是实现PBR能量守恒的核心。比如：$G(\vec{n}, \vec{v}, k)=\frac{\vec{n}*\vec{v}}{(\vec{n}*\vec{v})(1-k)+k}$。

其中，k是针对粗糙度的重映射(Remapping)，取决于我们要用的是针对直接光照还是针对IBL光照的几何函数。

$$
k_{direct}=\frac{(\alpha+1)^2}{8}\\
k_{ibl}=\frac{\alpha^2}{2}
$$

### Linear Space and Tone Mapping

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
