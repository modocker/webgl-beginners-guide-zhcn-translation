---
title: WebGL Beginner's Guide
tags: WebGL 初学者指南, Diego Cantor, Brandon Jones, hjlld 
grammar_cjkRuby: true
---

# WebGL 起步

> 在 2007 年，一位名叫  Vladimir Vukicevic 的美籍塞尔维亚裔软件工程师，
> 开始对即将到来的 HTML`<canvas>` 标签进行实现 OpenGL 的原型工作，他称之为“Canvas 3D”。
> 在 2011 年 3 月，他的工作指引了Khronos Group 这个 OpenGL 背后的非营利性组织去创建 WebGL： 
> 一个使互联网浏览器有能力去访问图形处理器（GPU）的标准。

WebGL 原本是基于 OpenGL ES 2.0（ ES 代表 Embedded Systems，即嵌入式系统）的， 这个特殊的 OpenGL 标准被广泛应用于大量设备，如苹果的 iPhone 和 iPad。但是随着标准的逐渐深入发展，WebGL 已经变成了一个以为跨平台系统和设备提供便携性为目标的独立标准。其基于网页和实时渲染的特性，为例如游戏、科学可视化、医学图像等网页 3D 环境打开了新的大门。另外，由于网页的普适性，这些 3D 应用也可以运行在例如智能手机和平板电脑等移动设备上。不管你是想要开始制作你的第一个网页 3D 游戏，还是虚拟美术馆，或者是将实验中的数据可视化，或者是其他什么 3D 应用，第一步永远是先确保你的环境设置。

在本章中，你将：

 - 理解 WebGL 应用的架构
 - 设置绘制区域（canvas）
 - 测试浏览器的 WebGL 能力
 - 理解 WebGL 作为状态机的运作原理
 - 修改 WebGL 中的变量观测场景变化
 - 载入并调试一个功能完备的场景

## 系统需求

作为一个基于网页的 3D 图形 API，WebGL 不需要任何安装过程。你可以使用任何支持 WebGL 的浏览器，包括但不限于：

- Mozilla Firefox 4.0+
- Google Chrome 11+
- Sarari OSX 10.6+ / iOS 8.0+
- Opera 12+
- Microsoft Edge 
- Micorsoft Internet Expolorer 11 （不推荐做开发使用）

如果想知道当前环境是否支持 WebGL 可以访问： http://get.webgl.org

几个有用的链接：

- https://caniuse.com/#feat=webgl 查看 WebGL 在各浏览器版本的支持情况
- https://caniuse.com/#feat=webgl2 查看 WebGL 2.0 在各浏览器版本的支持情况
- http://webglreport.com/ 查看当前环境中 WebGL 的实现能力
- https://webglstats.com/ 查看 WebGL 各项特性在市场中的实现数据

## WebGL 提供了哪种方式的渲染？

WebGL 作为一个 3D 图形接口使得浏览器具备以标准和高效的方式渲染 3D 场景的能力。根据维基百科的说法，渲染（Rendering）是将模型通过计算机程序方法生成为图像的过程。因为是通过计算机执行的过程，所以有很多不同的方法可以实现这种图像生成。

首先我们需要分辨的是，我们是否使用了任何特殊的图形硬件。首先我们要讨论的是软件渲染（Software-based Rendering），在这种渲染模式下，所有渲染 3D 场景所需要的计算都通过计算机的主处理器，也就是 CPU 来完成；而与之对应的另外一种渲染方法是硬件渲染（Hardware-based Rendering），在这种渲染模式下，由 GPU 来实现所有 3D 图形的实时运算。从技术角度看，硬件渲染比软件渲染更为高效，因为所有的计算有专门的硬件来完成。同时，软件渲染拥有更好的兼容性，可以作为在缺少硬件支持时的良好回滚。

其次我们要分辨的是，我们的渲染进程是在本地完成的还是在远程完成的。当需要渲染的图像太过复杂的时候，通常渲染器是运行在远程的。例如 3D 动画电影通常都是由数量巨大的专用服务器来进行复杂场景渲染的。我们称这种渲染为服务端渲染（Server-based Rendering）。与之相对的则是在本地计算的客户端渲染（Client-based Rendering）。

WebGL 通常都是作为客户端渲染的，尽管需要渲染的场景文件和某些元素是从服务器上下载的，但将模型渲染为图像的整个计算过程是在本地发生的，并使用客户端的硬件设备。

与其他类似的技术（Java 3D、Flash、Unity Web Player 等）相比，WebGL 有如下的优势：

- **使用 Javascript 编程**： Javascript 是天生面对网页浏览器和网页开发者的。相对其他的浏览器插件技术，使用 Javascript 可以更方便的操控 DOM 元素，并完成 DOM 元素间的通讯。因为 WebGL 使用 Javascript 编程，所以它更容易与其他 Javascript 框架或库整合并协同工作。

- **自动内存管理**：和它的近亲 OpenGL 或其他需要手动分配销毁内存的技术不同的是，WebGL 不需要手动管理内存。它遵循 Javascript 中的内存管理和运作方式，当内存不再需要时，浏览器的 Javascript 引擎会自动回收。这种简单的编程模式大大降低了代码量，使得代码更加清晰易读。

- **普适性**：感谢技术进步，浏览器会把 WebGL 带到任何安装的环境，例如智能手机、平板电脑和智能电视，甚至 XBOX 等游戏主机设备。

- **性能**：由于可以直接访问图形硬件，在大部分情况下，WebGL 的性能都可以比肩独立应用。WebGL 标准本身并没有问题，是 Javascript 语言的孱弱，导致了 WebGL 不能像本地编程一样，将 AAA 级游戏或复杂的 3D 应用带入到网页，目前出现的 Web Assembly 技术被认为是解决这一问题的希望。

- **无需编译**：因为 WebGL 使用 Javascript 编程，因此在执行前你无需进行任何编译。这使得修改应用并观测结果变化变得非常容易。但是需要说明的是，着色器代码依然需要在 GPU 中编译，但这是发生在 GPU 中的，而不是浏览器中的。

## WebGL 应用的架构

和任何的 3D 图形 API 一样，在 WebGL 中要想表现一个场景，你必须使用一些组件。本书的前四章将会涵盖这些基本元素。从第五章开始，我们将会讨论一些非必须的元素，例如颜色、纹理等高级话题。

我们所说的基本元素是指：

- **画布（`<canvas>`）**：它是场景渲染前的占位符，所有 WebGL 的绘制都会被限制在画布之内。`<canvas>` 是一个标准的 HTML5 DOM 元素，可以使用 Javascript 访问。

- **物体**：它是组成场景的 3D 实体。这些实体由三角形组成。在第二章中，我们将讨论 WebGL 是如何绘制几何体的。我们将使用WebGL Buffer 来储存几何体数据，并研究 WebGL 是如何使用这些 Buffer 将物体渲染到场景中的。

- **光照**：如果没有光，3D 世界将不会被照亮。我们将在第三章中讨论光照。我们将学习 WebGL 如何使用着色器在场景中建立光照模型。我们将会探讨物体如何根据物理法则去反射或吸收光照，我们还会讨论不同的光照模型。

- **相机**：画布只是 3D 世界的视口，我们要通过相机去探索穿越 3D 场景。在第四章中我们将会学习并理解不同的矩阵操作，以此来实现透视效果。我们还将学习如何把这些操作封装成相机。

本章我们先来探讨上述列表中的第一个基本元素 —— 画布。在接下来的章节中，我们讲学习如何创建一块画布并取得 WebGL 上下文。

## 创建 HTML5 画布

让我们来创建一个网页并添加一个 HTML5 `<canvas>` 元素。`<canvas>` 是一个矩形的元素，用于绘制 3D 场景。

``` xml
<!DOCTYPE html>
<html>
<head>
 <title> WebGL Beginner's Guide - Setting up the canvas </title>
 <style type="text/css">
 canvas {border: 2px dotted blue;}
 </style>
</head>
<body>
<canvas id="canvas-element-id" width="800" height="600">
Your browser does not support HTML5
</canvas>
</body>
</html> 
```

在这段代码中，我们创建了一个网页，并在其中添加了一个 `<canvas>` 元素，为了视觉上的清晰，我们使用 CSS 为这个 `<canvas>` 元素增加了一个边框样式，这样你可以清晰的分辨出 `<canvas>` 在哪，同时这也体现了 `<canvas>` 作为一个标准的 HTML5 DOM 元素的特性，它可以被施加任何其他的网页魔法，和谐地嵌入到网页整体氛围中。

我们还给 `<canvas>` 设置了一个 `id`，这样我们就可以通过 Javascript 的 DOM 操作找到它。

`<canvas>` 还有两个属性，分别是宽和高。当宽和高没有被设置时，浏览器通常会创建一个 300 x 150 的 `<canvas>`。

`<canvas>` 的宽和高，以及其 CSS 中的宽和高，通常是非常容易迷惑开发者的两处设置，尤其是在移动设备或高分屏下，如果设置不当会造成失真或其他意外发生。

在 WebGL 中针对画布有四个尺寸，分别是：

- **drawingBuffer 的尺寸**：由 `gl.drawingBufferWidth` 和 `gl.drawingBufferHeight` 获得。这是两个只读属性，WebGL 上下文将会自动将 `<canvas>` 标签中的 `width` 和 `height` 属性传入。drawingBuffer 决定了 GPU 中绘制的缓冲尺寸。

- **视口的尺寸**：由 `gl.viewport(x, y, width, height)` 函数设置，其中 `x` 代表视口左下角的水平坐标，`y` 代表视口左下角的垂直坐标，`x` 和 `y` 共同决定了视口的起始坐标点，`width` 和  `height` 代表视口的宽高。

- **`<canvas>` 的属性尺寸**： 在 `<canvas>` 标签中设置，会被 WebGL 上下文传入并设置为 drawingBuffer 的尺寸。

- **`<canvas>` 元素 CSS 样式中的尺寸**：因为 `<canvas>` 是一个标准的 HTML5 DOM 元素，因此可以像设置其他元素样式一样去设置 `<canvas>` 的 CSS 样式，它决定了 `<canvas>` 在网页中最终呈现的视觉尺寸。

**通常**来说，这几个尺寸应当保持一致，才可以绘制出没有错位、失真的 3D 场景。

一般的解决方案是对 `gl` 添加一个缩放函数，使得 `gl` 初始化时调用此函数，并捕捉页面的 `onresize` 事件，使用 `canvas.clientWidth` 和 `canvas.clientWidth` 获得 `<canvas>` 元素真实的视觉尺寸，然后及时对视口和 drawingBuffer 进行调整。使得视口、drawingBuffer 和 `<canvas>` 的视觉尺寸保持一致。

在高分屏的情况下，我们还必须考虑 `window.devicePixelRatio` 属性，并用它计算出真实的像素比例。一般来说，通常 3D 应用会在高分屏下渲染实际数量较少的像素，然后由 GPU 去缩放他们；随着 GPU 技术的发展和硬件配置的提高，在桌面平台许多游戏也开始提供超过实际像素数量的渲染选项，例如将 4K 图像渲染到 1080p 的显示器上，以提高图像的显示效果。但是在 WebGL 中在高分屏中渲染实际像素数量并不是一个好主意，如果 `window.devicePixelRatio = 2`，意味着 WebGL 将渲染 4 倍于原本的像素个数，这将大大降级性能。 

关于 WebGL 尺寸的讨论可以参考： https://webglfundamentals.org/webgl/lessons/zh_cn/webgl-resizing-the-canvas.html 。

当然，在使用例如 three.js 、babylon.js、playcanvas 等成熟的 WebGL 框架时，框架通常会自动帮你处理这些事情，你甚至无需手动设置 `<canvas>`的宽和高，还可以肆意的使用 CSS 来修改 `<canvas>` 的外观，也无需担心高分屏。

这里有一个流传多年的小技巧，就是说在硬件设备较差或性能敏感的情况下，可以创建一个尺寸较小的 `<canvas>` 例如 10 x 10，然后使用 CSS 缩放成 100 x 100，可以大量降低 3D 渲染带来的性能降低。原因是现代浏览器在处理 CSS 时，也会使用 GPU 运算，因此效率和效果会比想象的要好。

## 获取 WebGL 上下文

WebGL 上下文本质上是一个 Javascript Object 对象，通过 WebGL 上下文我们可以获取到所有 WebGL 函数和属性，即 WebGL API。

下面我们将尝试创建一个 Javascript 函数来检测是否可以在一个 `<canvas>` 上获得 WebGL 上下文。注意，所有 WebGL API 是内置于浏览器中的，你无需引入其他任何第三方库来使用这些函数或属性。

接下来我们将修改上一章节的代码，通过获取上下文的方式来检测浏览器是否支持 WebGL，我们需要在页面被加载完毕后调用这个函数，因此我们将使用 DOM 元素的 `onload` 事件。

``` javascript
window.onload = function () {
	var canvas = document.getElementById('canvas-element-id')
	var gl = canvas.getContext('webgl') || canvas.getContext('experimental-webgl')
	if (gl == null) {
		alert('WebGL NOT Supported!')
	} else {
		alert('WebGL  Supported!')
	}
}
```

代码非常简单，这里要讨论的有两个话题，分别是 WebGL 的上下文描述和使用上下文来判断是否支持 WebGL 是否可行。

首先，从 WebGL 的诞生之初，出现了各种各项的上下文描述，代码中使用了两个，分别是 `webgl` 和 `experimental-webgl`。在早期还有 `webkit-3d`、`moz-webgl` 等形形色色的上下文描述，在 WebGL 2.0 时代，又出现了  `webgl2` 和 `experimental-webgl2`。为了保证兼容性建议分别判断 WebGL 1.0 和 WebGL 2.0，在判断时同时使用 `webgl` 和 `experimental` 前缀。另外，`getContext()` 函数还接受第二个参数，作为初始化 WebGL 的选项，详情可以参考 [MDN](https://developer.mozilla.org/en-US/docs/Web/API/HTMLCanvasElement/getContext)。

其次，使用获取 WebGL 上下文来判断当前环境是否支持 WebGL 或 WebGL 2.0 是一个防君子不防小人的天真想法。尤其是在国内，在这么多年里，译者遇到无数号称支持 WebGL 的假环境，尤其出现在品牌泛滥的国内换壳浏览器中，以及各种手机 App 的 webview 中。

例如在微信小游戏的文档中明确指出，微信小游戏提供了 WebGL 1.0 的完整实现，根本没有提及 WebGL 2.0，但是实际上当我尝试去获取 WebGL 2.0 的上下文时，意外发生了，微信小游戏真的返回了一个 WebGL 2.0 的上下文，让引擎一脸懵逼，后续的渲染和逻辑全都乱了。

另外一个例子是，我曾经在一台较古老的华为手机（Android 5.0.1）上安装最新版本的微信后，由于微信自带的 X5 引擎，使得微信中的 webview 获得了 WebGL 2.0 的支持，但实际上例如浮点纹理和 color_buffer 等诸多 WebGL 2.0 的基本特性都是不可用的。

所以在这些不严谨的 WebGL 实现中，通过获得上下文去判断 WebGL 能力，是一种非常不靠谱的手段。这本身是由于国内浏览器或 webview 内核开发者的偷懒，给 WebGL 开发者带来了很多的不便。

可行的解决方案是，对于浏览器开发者来说，WebGL 有一套很好的[一致性测试套件](https://www.khronos.org/registry/webgl/conformance-suites/2.0.0/webgl-conformance-tests.html)，浏览器开发者应当在发布前，在多种硬件环境下运行 WebGL 一致性测试，确保 WebGL 实现的准确度；对于 WebGL 开发者来说，应当使用更为详细的检测手段，例如在固定大小的的 `<canvas>` 中绘制一个带纹理的简单几何体，然后用 `gl.readPixels()` 函数读出，与标准样本图片进行对比，并在过程中 `catch` 错误，确保真的能绘制出一个与标准样本图片相符的场景，而对于其他更加深入的特性，例如是否支持浮点纹理等，应当直接做代码测试。

## WebGL 是状态机

关于状态机原理和状态机编程思想是一个非常专业的计算机科学知识，在本书中将会多次提及 WebGL 的状态机机制。

WebGL 的上下文可以被理解为一个状态机，一旦你修改了其中的一个属性，这个修改将会持续保持到你再次对这个属性做出修改。任何时候你都可以查询到这些属性的状态，所以你可以知道 WebGL 上下文当前的状态。让我们来看代码。

``` xml
<html>
<head>
	 <title> WebGL Beginner's Guide - Setting WebGL context attributes</title>
	 <style type="text/css">
	 canvas {border: 2px dotted blue;}
	 </style>
	 
	 <script>
		var gl = null;
		var c_width = 0;
		var c_height = 0;
		
		window.onkeydown = checkKey;
		
		function checkKey(ev){
		  switch(ev.keyCode){
			case 49:{ // 1
				gl.clearColor(0.3,0.7,0.2,1.0);
				clear(gl);
				break;
			}
			case 50:{ // 2
				gl.clearColor(0.3,0.2,0.7,1.0);
				clear(gl);
				break;
			}
			case 51:{ // 3
				var color = gl.getParameter(gl.COLOR_CLEAR_VALUE);
				
				//Don't get confused with the following line. 
				//It basically rounds up the numbers to one 
				//decimal cipher just for visualization purposes.
				alert('clearColor = (' + Math.round(color[0]*10)/10 + ',' +Math.round(color[1]*10)/10+','+Math.round(color[2]*10)/10+')');
				window.focus();
				break;
			}
		  }
		}
		
		function getGLContext(){
			var canvas = document.getElementById("canvas-element-id");
			if (canvas == null){
				alert("there is no canvas on this page");
				return;
			}
				
			var names = ["webgl", "experimental-webgl", "webkit-3d", "moz-webgl"];
            var ctx = null;
            for (var i = 0; i < names.length; ++i) {
                try {
                    ctx = canvas.getContext(names[i]);
                } 
                catch(e) {}
                if (ctx) break;
            }

			if (ctx == null){
				alert("WebGL is not available");
			}
			else{
				return ctx;
			}
		}
		
		function clear(ctx){
			ctx.clear(ctx.COLOR_BUFFER_BIT);
			ctx.viewport(0,0,c_width, c_height);
		}
		
		function initWebGL(){
			gl = getGLContext();
			
		}
	 </script>
</head>
<body onLoad='initWebGL()'>
<canvas id="canvas-element-id" width="800" height="600">
Your browser does not support the HTML5 canvas element.
</canvas>
</body>
</html>
```

在这段代码中我们编写了如下几个函数：

| 函数名         | 说明                                                                                                                           |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| `checkKey`     | 辅助函数，用于截取键盘事件，并根据键值执行相应代码。                                                                           |
| `getGLContext` | 创建 WebGL 上下文。                                                                                                            |
| `clear`        | 用某种颜色清空 `<canvas>`，这是 WebGL 的一个属性。像之前提到的，WebGL 是一个状态机，因为他将会保持这个颜色直到颜色被再次改变。 |
| `initWebGL`    |  初始化 WebGL，并将其上下文作为全局变量返回。                                                                 |

运行页面后，当按 <kbd>1</kbd> 键时，`<canvas>` 变为绿色；当按  <kbd>3</kbd> 键时，将会弹出提示框显示当前使用的颜色值；`<canvas>` 将会持续保持绿色，直到我们调用 `gl.clearColor` 再次更改颜色，当按  <kbd>2</kbd> 键时，我们会将将此颜色更改为蓝色。

在这个示例中，我们调用了 `gl.clearColor` 函数来改变了画布的颜色状态，相应的，我们调用 `gl.getParameter(gl.COLOR_CLEAR_VALUE)` 来获取当前的颜色状态。

在本书中我们将会看到很多类似的函数，一边是设置状态，一边是通过相应参数获取状态。

## 使用上下文来获取 WebGL API

另外需要指出的是，所有的 WebGL 函数都是通过 WebGL 上下文来获取到的，在这个实例中我们用 `gl` 这个变量来储存 WebGL 上下文，因此所有的 WebGL API 都是通过这个变量来运行的。

## 总结

在本章中，我们归纳了 WebGL 应用所必备的四个组件，分别是：画布、物体、光照和相机。

我们学习了如何在网页中创建 `<canvas>` 元素并设置它的 ID 和尺寸，然后我们又创建了 WebGL 上下文。我们初步了解了 WebGL 作为状态机的运行原理，并通过 `gl.getParameter` 来获取当前状态。

在下一章中，我们讲学习如何在 WebGL 场景中定义、载入和渲染 3D 物体。