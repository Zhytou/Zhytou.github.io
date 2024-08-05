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

OpenGL

使用OpenGL渲染的流程其实和画油画类似，包括以下几个步骤：

- 创作油画的第一步是摆放物品/模型。类似的，渲染的第一步则是准备好待渲染模型的顶点数据，包括：顶点、法线以及纹理坐标等，并在稍后使用顶点缓存对象（Vertex Buffer Object，VBO）将其传入给GPU。
- 创作油画的第二步是摆好画架。类似的，渲染的第二步则是设置好相机位置，进行视图转换（View Transformation）。同时，还要创建顶点

### The Rendering Equation

## 3 Real-Time Shadows(1)

### Shadow Mapping

## 5 Real-Time Environment Mapping

### Recap: Image-Based Lighting
