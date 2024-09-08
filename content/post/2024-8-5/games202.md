---
title: "GAMES202 学习笔记"
date: 2024-08-05T18:26:29+08:00
draft: false
---

## 1 Introdution

**Real-Time High Quality Rendering**:

- 实时性：30 frame per second（对于VR/AR，要求可能到90FPS）；
- 高质量：尽可能保证结果真实，即对错误容忍度低；
- 渲染：

**4 Different Parts of Real-Time Rendering**：

- 阴影
- 全局光照
- 基于物理的着色
- 实时光线追踪

**Motivation of Real-Time Rendering**：

目前，计算机图形学中的技术已经支持生成一些极逼真的图像，但这些方法大多耗时过长，比如光线追踪等。因此，出现了一系列针对实时渲染的研究，它们的目的就是牺牲一小部分准确性，来获取较高质量的快速渲染。

**Evolution of Real-Time Rendering**：

- 20年前：图形硬件（SGI/GPU）出现，大部分重点放在几何图形和纹理映射上，同时出现了一些调增强真实感的技术：阴影映射、累积缓冲区等。
- 10年到20年前：着色器出现（可编程图形硬件），更多复杂的图形渲染技术出现，比如：环境光、软阴影等
- 目前：VR/AR

**Technological and Algorithmic Milestones**：

- 20年前：可编程图形硬件（着色器）
- 15年前：基于预计算的渲染方法
- 8-10年前：交互式光线追踪

## 2 Recap of CG Basics

### Graphics Pipeline

### OpenGL

**A Set of API**：

OpenGL是一组可编程渲染管线API的封装。它的特点包括：

- 跨平台；
- 很多替代技术，比如：DirectX和Vulkan；
- C风格的。

**Analogy: Oil Painting**：

使用OpenGL渲染的流程其实和画油画类似，包括以下几个步骤：

- 创作油画的第一步是摆放物品/模型。类似的，渲染的第一步则是准备好待渲染模型的顶点数据，包括：顶点、法线以及纹理坐标等，并在稍后使用顶点缓存对象（Vertex Buffer Object，VBO）将其传入给GPU。
- 创作油画的第二步是摆好画架。类似的，渲染的第二步则是设置好相机位置，进行视图转换（View Transformation）。同时，还要创建顶点

### The Rendering Equation

**BRDF**：

双向反射分布函数（Bidirectional Reflectance Distribution Function，BRDF）是真实感图形学中最核心的概念之一，它描述的是物体表面将光能从任何一个入射方向反射到任何一个视点方向的反射特性，即入射光线经过某个表面反射后如何在各个出射方向上分布。

**Cook-Torrance BRDF**：

Cook-Torrance BRDF是一种目前最流行一种BRDF反射模型，它包含漫反射和镜面反射两个部分：$f_r=k_df_{lambert}+k_sf_{cook-torrance}$。其中，$f_{lambert}=\frac\rho\pi$，而$f_{cook-torrance}=\frac{DFG}{4(\vec{w_o}*\vec{n})(\vec{w_i}*\vec{n})}$。

## 3 Real-Time Shadows

### Shadow Mapping

阴影贴图是目前最主流的阴影生成算法。这主要得益于它算法直观，并且能够充分利用现代硬件的光栅化能力。具体来说，它的核心思路基于一个假设，即对于指定光源来说，场景中某个点是否被其照亮，取决于从光源的视角看去，这个点是否可见。

**Two-Pass Algorithmn**：

因此，该算法被分成了两步：

- 从光源的视角出发，绘制整个场景（平行光用正交投影，点光用透视投影），生成深度图，即所谓的阴影贴图；
- 从摄像机视角出发，重新绘制场景，并根据光源投影矩阵的逆矩阵，将世界坐标空间变换回光源的投影空间，找出对应投影空间UV坐标以及投影空间内的深度d。

此时，就可以使用光源投影空间的UV坐标和阴影贴图，得到深度z。比较深度z和深度d，若d>z，则当前位置被遮挡，处于阴影内；反之，则未被遮挡。

可见，阴影贴图是一个完全在图像空间中的算法。它的优点是一旦贴图已经生成，则其就可以作为场景中的几何表示(场景中所有位置在光源处的深度)，而不用再去获取场景中的其他信息。而缺点则是算法会产生自遮挡和走样现象。

**Simple Implementation with GLSL**：

了解了阴影贴图的流程之后，我们可以简单实现它。比如：Phase 1中预生成阴影纹理的片元着色器如下。

```glsl
#ifdef GL_ES
precision mediump float;
#endif

uniform vec3 uLightPos;
uniform vec3 uCameraPos;

varying highp vec3 vNormal;
varying highp vec2 vTextureCoord;

vec4 pack (float depth) {
    // 使用rgba 4字节共32位来存储z值,1个字节精度为1/256
    const vec4 bitShift = vec4(1.0, 256.0, 256.0 * 256.0, 256.0 * 256.0 * 256.0);
    const vec4 bitMask = vec4(1.0/256.0, 1.0/256.0, 1.0/256.0, 0.0);
    // gl_FragCoord:片元的坐标,fract():返回数值的小数部分
    vec4 rgbaDepth = fract(depth * bitShift); //计算每个点的z值
    // Cut off the value which do not fit in 8 bits
    rgbaDepth -= rgbaDepth.gbaa * bitMask; 
    return rgbaDepth;
}

void main(){
  gl_FragColor = pack(gl_FragCoord.z);
}
```

Phase 2中显示的片元着色器如下：

```glsl
float unpack(vec4 rgbaDepth) {
  const vec4 bitShift = vec4(1.0, 1.0/256.0, 1.0/(256.0*256.0), 1.0/(256.0*256.0*256.0));
  return dot(rgbaDepth, bitShift);
}

float useShadowMap(sampler2D shadowMap, vec4 shadowCoord){
  float d = unpack(texture2D(shadowMap, shadowCoord.xy));
  if (shadowCoord.z > d + EPS) {
    return 0.0;
  } else {
    return 1.0;
  }
}

void main(void) {

  //vPositionFromLight为光源空间下投影的裁剪坐标，除以w结果为NDC坐标
  vec3 shadowCoord = vPositionFromLight.xyz / vPositionFromLight.w;
  //把[-1,1]的NDC坐标转换为[0,1]的坐标
  shadowCoord = (shadowCoord + 1.0) / 2.0;

  // 计算是否为阴影（因为是硬阴影，所有返回值仅为0或1）
  float visibility = useShadowMap(uShadowMap, vec4(shadowCoord, 1.0));
  // 计算颜色
  vec3 phongColor = blinnPhong();

  gl_FragColor = vec4(phongColor * visibility, 1.0);
}
```

**Issues in Shadow Mapping**：

阴影失真（Shadow Acne）是阴影贴图中最容易出现的问题。它表现为在阴影区域出现间隔的条纹，如下图所示。

![阴影失真](https://i-blog.csdnimg.cn/blog_migrate/c4d0c52217b34561520e3340a93f7200.png)

出现这种现象的原因在于ShadowMap中的每个texel只能记录一个深度值，但一个像素内的不同位置可能会有不同的深度值（即，试图用离散的纹理去记录连续的信息）。因此，当实际场景的深度值大于ShadowMap上记录的深度值时，就会出现Shadow Acne现象。

一个简单通用的解决方案是在比较前给d加上一个固定偏移。但是若偏移选取过大，又会产生所谓的阴影悬空（Peter-Panning）问题，如下图所示。

![阴影悬空](https://pic2.zhimg.com/80/v2-8b7a421a28ac57a3b779cb824f1e0cb9_720w.webp)

改进的方法则是添加自适应偏移的方案，即基于斜率去计算当前深度要加的偏移（Slope Scale Depth Bias）。

另外，阴影走样（Shadow Aliasing）也是阴影贴图中容易出现的问题。比如

![阴影走样](https://i-blog.csdnimg.cn/blog_migrate/08e1b652b0eb13a489399f583989c68c.png)

其原因则是阴影贴图是一个大小不足，造成多个片段对应一个纹理像素。工业界的解决办法是，不同位置采用不同分辨率的shadow map，或者是动态分辨率之类的技术，也可以直接用PCF生成软阴影。

除此之外，阴影贴图还存在纹理利用不足的问题。因为相机和光源视角不同，所以从光源视角光栅化后的每个像素投影到屏幕空间后，对应的区域大小也不相同，进而引发靠近相机位置的阴影贴图精度不够，而远离相机位置的阴影贴图精度又过高的问题。改善这种利用率不足的方法包括：级联阴影贴图和Sample Distribution Shadow Map。

### Percentage Closer Filtering

百分比渐进滤波（Percentage-Closer Filtering，PCF）是一种滤波工具。它可以改善阴影的平滑度和真实感，减少硬边缘和锯齿效果。

PCF并非在直接在最后生成的结果上做模糊，或是对Phase1计算得到阴影贴图进行模糊，而是在做阴影判断时进行滤波操作从而生成更合理的visibility。因为已经出现走样即高频重叠后，再去进行模糊是无法消除重叠的；而对阴影贴图进行模糊并不能改善visibility返回值非0即1的情况。

具体来说，其原理就是计算visibility时，在原阴影贴图坐标一定范围中，选择若干随机点；将这些点的深度值，和原点的深度值作比较，最后将其加和取平均即可。其在GLSL中的实现如下：

```GLSL
float PCF(sampler2D shadowMap, vec4 coords, float filterSize) {
  float visibility = 0.0;

  poissonDiskSamples(coords.xy);
  for (int i = 0; i < NUM_SAMPLES; i++) {
    // 偏移量 = 0到1随机变量 * 归一化后的采样半径
    vec4 offset = vec4(poissonDisk[i], 0, 0) * filterSize;
    // 采样点位置 = 实际位置 + 偏移量
    float d = useShadowMap(shadowMap, coords + offset);
    if (d + EPS > coords.z) {
      visibility = visibility + 1.0;
    }
  }
  // 取平均
  return visibility / float(NUM_SAMPLES);
}
```

### Percentage Closer Soft Shadow

如下图所示，在真实世界中，笔尖的阴影十分锐利，而笔杆的阴影会软一些。

![钢笔的阴影](https://i-blog.csdnimg.cn/blog_migrate/99b100153b711cf71511864ea50e5462.png)

由此，我们可以得出结论：

- 承载阴影的物体表面点与阴影产生物的距离越小，阴影越硬。
- 阴影的软硬与光源距离无关，与遮挡物距离有关

![软阴影面积变化示意图](https://i-blog.csdnimg.cn/blog_migrate/7c25e9e78dca72b3a5708523fcd85c50.png)

进一步，我们可以根据相似三角形定理从上述示意图中推导出一个公式，这也是PCSS的核心。

$$
\frac{W_{Penumbra}}{W_{Light}}=\frac{d_{Receiver}-d_{Blocker}}{d_{Blocker}}
$$

其中，$W_{Penumbra}$表示软阴影大小；$W_{Light}$表示光源大小；$d_{Receiver}$表示光源到片元的距离；$d_{Blocker}$表示光源到遮挡物的距离。

百分比渐进软阴影（Percentage Closer Soft Shadow, PCSS）技术是对PCF的进一步改进，它能实现更逼真的阴影效果，即阴影的软硬会随承载阴影的物体表面与阴影产生物的距离变化。换句话说，它能根据上述原理针对不同片元生成自适应软阴影大小，也即上述PCF实现中使用的采样范围。其GLSL实现如下：

```GLSL
float findBlocker(sampler2D shadowMap, vec2 uv, float zReceiver, float searchRange) {
  float depthSum = 0.0;
  int blockerNum = 0;

  poissonDiskSamples(uv);
  for (int i = 0; i < NUM_SAMPLES; i++) {
    float d = unpack(texture2D(shadowMap, uv + poissonDisk[i] * searchRange));
    if (d + EPS < zReceiver) {
      depthSum = depthSum + d;
      blockerNum = blockerNum + 1;
    }
  }

  if (blockerNum == 0) {
    return 0.0;
  }

  return depthSum / float(blockerNum);
}

float PCSS(sampler2D shadowMap, vec4 coords, float searchRange){

  // STEP 1: avgblocker depth
  float avgDepth = findBlocker(shadowMap, coords.xy, coords.z, searchRange);

  // STEP 2: penumbra size
  if (avgDepth == 0.0) {
    return 1.0;
  }
  float penumbraSize = (coords.z - avgDepth) / avgDepth * LIGHT_SIZE;

  // STEP 3: filtering
  float visibility = PCF(shadowMap, coords, penumbraSize);

  return visibility;
}
```

### Variance Soft Shadow Mapping

PCSS、PCF 的算法都需要多重采样，尤其 PCSS 需要两个多重采样（第一步的Blocker Search和第三步的PCF），这使得算法速度较慢。

为了避免多重采样的计算，Variance Soft Shadow Mapping（VSSM）假定一定范围内的深度的分布符合正态分布（Normal Distribution） ，那么只要知道该段范围的均值E 、方差Var，就能先得到该范围的概率密度函数PDF。进而通过该概率密度函数的积分——累计分布函数CDF快速计算出该范围中有多少比例大于某个指定深度。

### Cascaded Shadow Mapping

## 4 Real-Time Environment Mapping

### Recap: Image-Based Lighting

基于图像的光照(Image Based Lighting, IBL)

### Split Sum Approximation
