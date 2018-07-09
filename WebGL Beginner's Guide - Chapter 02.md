---
title: WebGL Beginner's Guide - Chapter 02
tags: WebGL 初学者指南, Diego Cantor, Brandon Jones, hjlld 
grammar_cjkRuby: true
---

# 几何体渲染

> WebGL 在渲染物体是遵循“分治法（Divide andconquer）”原则。复杂的几何体被分解成三角形、线和点这三种基元。然后，每个几何意义上的基元在 GPU 种通过一系列步骤 —— 也就是所谓的“渲染管线” —— 进行并行处理，最终创建出显示在画布上的场景。

在本章中，我们将：

- 理解 WebGL 如何定义和处理几何信息。
- 讨论与操作几何体相关的 API 方法。
- 讨论如何使用 JSON 定义、储存和载入几何体及其原因。
- 继续讨论 WebGL 作为状态机的工作原理，以及如何设置、读取与操作几何体有关的属性状态。
- 尝试创建和载入不同的几何体模型。

## 顶点和索引

与复杂程度相当高、点线面特别多的几何体相比，WebGL 处理几何体使用了一种相对简单标准的方式。呈现 3D 物体的几何结构需要两种基本的数据类型，分别是：顶点和索引。

顶点是用于定义 3D 物体角上的点。每个顶点有三个浮点数分别代表其 x, y, z 的坐标。和 OpenGL 不同，WebGL 没有提供任何的 API 方法去向渲染管线传输顶点，因此我们需要用 Javascript 数组重写所有顶点相关的代码，然后构建一个 WebGL 顶点缓存。

索引是指在给定的 3D 场景中每个顶点的数字标签。索引让我们可以告诉 WebGL 如何连接顶点以生成一个面。和顶点一样，索引也是储存在 Javascript 数组中然后用 WebGL 索引缓存传值给渲染管线的。

> 有两种 WebGL 缓存用于描述和处理几何体，包含顶点数据的缓存被称为 Vertex Buffer
Objects（VBOs）。类似的，包含索引数据的缓存被称为 Index Buffer Objects（IBOs）


 