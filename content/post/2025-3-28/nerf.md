---
title: "Nerf"
date: 2025-03-23T10:58:52+08:00
draft: false
---

## 渲染与反渲染

![inverse_render](https://zhytou.github.io/post/2025-4-11/inverse_render.png)

反渲染（Inverse Rendering）是计算机视觉和计算机图形学中的一个重要概念，旨在从观察到的图像中推断出场景的三维结构、光照条件和材质属性等信息。通过这些信息，我们可以重建场景的三维模型，或者生成新的视角下的图像。

反渲染的关键点包括：

- 由图像反渲染的得到形状（shape）、材质（material）和光照（lighting）如何表示？
- 如何根据这些信息进一步生成新的视角下的图像，即如何再次渲染（render）？

### 形状表征

点云（Point Cloud）：点云是一种稀疏的三维表示方法，由一组离散的点组成，每个点包含其在三维空间中的坐标和其他属性（如颜色、法线等）。点云通常用于表示复杂的三维形状，如扫描得到的物体表面。

网格（Mesh）：网格是一种更为常见的三维表示方法，由顶点、边和面组成。网格可以表示更复杂的形状，并且通常具有更高的渲染效率。网格的每个面通常是一个三角形或四边形，顶点之间通过边连接形成面。

占据场（Occupancy Field）是一种隐式的三维表示方法，通过一个函数来判断空间中某个点是否被占据。占据场通常用于表示稀疏的三维结构，如室内场景或城市环境。

### 外观表征

![apperance_representation](https://zhytou.github.io/post/2025-4-11/apperance_representation.png)

## Nerf

> [Representing Scenes as Neural Radiance Fields for View Synthesis](https://arxiv.org/pdf/2003.08934)

神经辐射场（Neural Radiancce Field，Nerf）是一种基于深度学习的3D场景表示与视图合成方法，能够从多视角2D图像中重建出高质量的三维场景，并生成任意视角下的逼真新视图。其核心思想是通过神经网络隐式地建模场景中每个3D点的颜色和密度分布，结合光线追踪技术实现物理真实的渲染效果。

## 3DGS

3D高斯泼溅（3D Gaussian Splatting，3DGS）是一种基于高斯点集的三维场景表示与渲染技术，其功能是通过优化三维高斯分布的位置、形状和颜色参数，实现高精度场景重建与实时新视角合成。

> [A Survey on 3D Gaussian Splatting](https://arxiv.org/pdf/2401.03890)
