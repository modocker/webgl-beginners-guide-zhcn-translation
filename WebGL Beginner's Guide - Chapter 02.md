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

顶点是用于定义 3D 物体角上的点。每个顶点有三个浮点数分别代表其 x, y, z 的坐标。和 OpenGL 不同，WebGL 没有提供任何的 API 方法去向渲染管线传输顶点，因此我们需要用 JavaScript 数组重写所有顶点相关的代码，然后构建一个 WebGL 顶点缓存。

索引是指在给定的 3D 场景中每个顶点的数字标签。索引让我们可以告诉 WebGL 如何连接顶点以生成一个面。和顶点一样，索引也是储存在 JavaScript 数组中然后用 WebGL 索引缓存传值给渲染管线的。

> 有两种 WebGL 缓存用于描述和处理几何体，包含顶点数据的缓存被称为 Vertex Buffer
Objects（VBOs）。类似的，包含索引数据的缓存被称为 Index Buffer Objects（IBOs）

在继续深入之前，让我们先来看一下 WebGL 的渲染管线大概是什么样子，而在这个架构中缓存数据又扮演了什么角色。

## WebGL 渲染管线概览

在这里我们将会看到一个简易版本的 WebGL 渲染管线。在接下来的章节中，我们将会讨论其中的更多细节。

![Diagram](./attachments/1531190440417.drawio.html)

让我们花一点时间来解释其中的每一个部分。

### 顶点缓存（VBO）
VBO 包含 WebGL 所需的描述渲染几何体的信息。就想之前提到的，顶点坐标在 WebGL 中通常以 VBO 的形式储存和处理。另外，例如顶点法线、颜色、纹理坐标等等，都可以使用 VBO 为模型来储存和处理。

### 顶点着色器（Vertex Shader）
在处理每个顶点是会调用顶点着色器。这个着色器将以“逐顶点（per-vertex）”的形式处理顶点坐标、法线、颜色和纹理坐标等数据。这些数据以属性（Attributes）的形式在顶点着色器中复现。每个属性都指向一个 VBO，从中将读取顶点数据。

### 片元着色器（Fragment Shader）
每三个顶点集合将定义一个三角形，每个三角形上中图形基元都需要被分配一个颜色，否则将会呈现为透明。

每个图形基元被称为一个片元（Fragment）。因为我们最终的渲染输出是显示在你的显示器上的，所以这些图形基元有个更普通的名字：像素。

片元着色器的主要职责是计算每个独立像素的颜色，如下图所示

![Diagram](./attachments/1531734136532.drawio.html)

### 帧缓冲（Framebuffer）
帧缓冲是一个二维的缓存，其中储存着经过片元着色器处理过的片元数据。一旦所有片元都被处理完成，一张 2D 图像就被组装完成并呈现在显示器中。

帧缓冲是渲染管线的最终终点。

### Attribute、Uniform、Varying
Attribute、Uniform 和 Varying 是着色器编程中三种不同类型的变量。

Attribute 是在顶点着色器中使用的输入变量。例如，顶点坐标、顶点颜色等等。因为对于每一个顶点都将会调用顶点着色器，所以每次顶点着色器被调用时，Attribute 都将发生变化。

Uniform 是在顶点着色器和片元着色器中都可以访问的输入变量。和 Attribute 不同，Uniform 在渲染循环中是一个敞亮。例如，灯光的位置。

Varying 用于从顶点着色器向片元着色器中传递数据。

好了，现在让我们开始创建一个简单的几何体。

## 在 WebGL 中渲染几何体
接下来我们将按照以下步骤在 WebGL 中渲染一个几何体：

1. 首先，我们要用 JavaScript 数组定义一个几何体。
2. 其次，我们要创建相应的 WebGL 缓存。
3. 第三，我们要把一个顶点着色器 Attribute 变量指向上一步中创建的用于储存顶点数据的 VBO。
4. 最后我们用 IBO 来完成渲染。

### 使用 JavaScript 数组定义几何体
让我们来看一下如何创建一个梯形。我们需要两个 JavaScript 数组，一个用于顶点，一个用于索引。

![Diagram](./attachments/1531735649842.drawio.html)

如上图所示，我们在顶点数组中依次放置了每个顶点的坐标，然后根据其绘制梯形的方式，将它们指向索引数组。也就是说，第一个三角形是由顶点 0, 1, 2 组装成的，第二个三角形是由顶点 1, 2, 3 组成的，最后一个三角形是由顶点 2, 3, 4 组成的。我们将会按照这个流程绘制所有可能的几何体。

### 创建 WebGL 缓存
在我们创建 JavaScript 数组用于定义顶点和索引后，我们就可以开始创建对应的 WebGL 缓存。让我们用另外一个更简单的示例来看下是如何创建的。在这个示例中，我们在 x-y 平面上绘制一个简单的正方形（即 z 坐标对于所有四个顶点都是 0）。

``` javascript
var vertices = [-50.0, 50.0, 0.0,
 -50.0,-50.0, 0.0,
 50.0,-50.0, 0.0,
 50.0, 50.0, 0.0];/* our JavaScript vertex array */
var myBuffer = gl.createBuffer(); /*gl is our WebGL Context*/
```

在上一章中，我们说过 WebGL 是作为一个状态机在运行的。现在，当 `myBuffer` 被当做当前绑定的 WebGL 缓存，也就是说，所有都后续缓存操作都将依赖于此缓存，直到它被解绑，或者另外一个缓存被绑定。我们通过下面这个函数进行绑定操作：

``` javascript
gl.bindBuffer(gl.ARRAY_BUFFER, myBuffer);
```

其中的第一个参数代表了我们要穿件的缓存类型，在这个参数里，我们有两个选择：

- `gl.ARRAY_BUFFER`：顶点数据
- `gl.ELEMENT_ARRAY_BUFFER`：索引数据

在这个示例中，我们要创建顶点坐标的缓存，因此我们使用了 `gl.ARRAY_BUFFER` 参数。对于索引数据，则要使用 `gl.ELEMENT_ARRAY_BUFFER`。

> WebGL 会一直访问当前绑定的缓存区查找数据，因此在进行任何其他操作时务必确认正确绑定了缓存。
> 如果 WebGL 没有找到当前绑定缓存，会抛出 `INVALID_OPERATION` 错误。

在绑定缓存后，我们需要将其中的数据传递给 WebGL。我们使用 `bufferData` 函数。

``` javascript
gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(vertices), gl.STATIC_DRAW);
```

在这个示例中，`vertices` 是一个储存顶点坐标的 JavaScript 数组。但是 WebGL 并不能直接接受 JavaScript 数组 作为 `bufferData` 函数的参数。相反，WebGL 使用类型化数组（typed array），这样做的原因是缓存数据可以直接被当做二进制数据来进行处理，以提升性能。

> 关于类型化数组（typed array）的标准可以访问： http://www.khronos.org/registry/typedarray/specs/latest/

WebGL 使用的类型化数组包括 `Int8Array`、`Uint8Array`、`Int16Array`、
`Uint16Array`、`Int32Array`、`UInt32Array`、`Float32Array` 和 `Float64Array`。

通过观察我们可以发现，顶点坐标可以是浮点数，但是索引永远是整型数。因此，在本书中，对于 VBO 我们使用 `Float32Array`，而对于 IBO 我们使用 `Uint16Array`。这两个类型也是你在 WebGL 每次渲染命令中可以使用的最大的类型化数组。

