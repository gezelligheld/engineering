可缩放矢量图形（svg）在放大或改变尺寸的情况下其图形质量不会有所损失，相比于 jpg 和 png 尺寸更小，但是复杂度越高渲染速度就会越慢。SVG 的绘制其实就是一个 SVG 标签，然后在标签内绘制你要绘制的内容

#### 属性

- viewBox="x y w h"：定义了 SVG 中可以显示的区域，示例如下，相当于放大了区域只显示 viewBox 部分

```js
<svg width="300" height="300" viewBox="0 0 100 100">
  <circle cx="100" cy="100" r="100" />
</svg>
```

- version：指明 SVG 的版本，目前只有 1.0 和 1.1 这两个版本

- xmlns：用于声明命名空间，用来区分同样名称的标签是 html 下的还是 svg 下的
- xmlns:xlink：表示前缀为 xlink 的标签和属性，应该由理解该规范的 UA 使用 xlink 规范 来解释

```js
<svg
  xmlns="http://www.w3.org/2000/svg"
  xmlns:xlink="http://www.w3.org/1999/xlink"
></svg>
```

#### 基本图形

```js
<svg width="300" height="300">
  // 圆形，cx、cy为圆的坐标，r为圆的半径
  <circle cx="100" cy="100" r="100" />
  // 矩形，x、y为矩形的起始点坐标，rx、ry为圆角x、y轴方向的半径，
  width、height为矩形的宽高
  <rect x="0" y="0" rx="5" ry="5" width="300" height="200" />
  // 椭圆，cx、cy为椭圆的坐标，rx为椭圆的x轴半径、ry为椭圆的y轴半径
  <ellipse cx="100" cy="100" rx="100" ry="50" />
  // 线，x1、y1为起点的坐标，x2、y2为终点的坐标
  <line x1="50" x2="50" y1="200" y2="50" style="stroke: #000000;" />
  //
  折线，points为点集数列，其中每个点都必须包含2个数字，一个是x坐标，一个是y坐标
  <polyline
    points="0 0, 20 40, 70 80, 100 90, 200 30, 250 50"
    fill="none"
    style="stroke: #000000;"
  />
  //
  多边形，points为点集数列，其中每个点都必须包含2个数字，一个是x坐标，一个是y坐标，默认闭合
  <polygon
    points="0 0, 20 40, 70 80, 100 90, 200 30, 250 50"
    fill="none"
    style="stroke: #000000;"
  />
</svg>
```

#### 路径

路径可以绘制任何图形，用属性 d 定义，通过命令 + 参数 的形式进行组合

- 直线命令

  - M（Move to）：把画笔移动到某个点
  - L（Line to）：从当前点绘制一条直线到某个点
  - H（Horizontal Line to）：从当前点绘制一条水平的直线
  - V（Vertical Line to）：从当前点绘制一条垂直的直线
  - Z（Close path）：闭合命令

```js
<svg width="300" height="300">
    <!-- 从起始点（50， 20）画一条到（250， 20）的直线 -->
    <path d="M50 20 L250 20" style="stroke: #000000;"/>
    <!-- 从起始点（50， 20）画一条X轴为250的水平直线 -->
    <path d="M50 20 H250" style="stroke: #000000;"/>
    <!-- 从起始点（50， 20）画一条Y轴为250的垂直直线 -->
    <path d="M50 20 V250" style="stroke: #000000;"/>
    <!-- 三角形>
    <path d="M50 20 H200 V200 Z" fill="none" style="stroke: #000000;"/>
</svg>
```

- 曲线命令

  - Q（Quadratic Bezier Curve to）：二次贝塞尔曲线
  - T（Smooth Quadratic Bezier Curve to）：延长二次贝塞尔曲线，即添加一个控制点
  - C（Curve to）：三次贝塞尔曲线
  - A（Elliptical Arc）：圆弧

```js
<svg width="300px" height="300px">
  <path d="M50 100 Q 175 200 300 100" fill="none" style="stroke: #ff0000;" />
  <path
    d="M50 100 Q 175 200 300 100 T 600 100 "
    fill="none"
    style="stroke: #ff0000;"
  />
  <path
    d="M50 50 C 100 100, 200 100, 250 50"
    fill="none"
    style="stroke: #000000;"
  />
  <path
    d="M10 100 50 100  A 30 50 0 0 1 150 100 L 200 100"
    fill="none"
    style="stroke: #ff0000"
  />
</svg>
```

#### 填充和轮廓

- fill：填充颜色
- fill-opacity：填充颜色透明度

- stroke：定义线条、文本或元素轮廓的颜色
- stroke-width：轮廓宽度
- stroke-opacity：轮廓透明度
- stroke-linecap：轮廓两端样式
- stroke-linejoin：轮廓连接处样式
- stroke-dasharray：定义轮廓为虚线
- stroke-dashoffset：虚线偏移量
- stroke-miterlimit：限制当两条线相交时交接处最大长度，配合 stroke-lineJoin：‘miter’使用

#### 文字

- 通过 text 标签可以在 svg 中添加文字

  - x 和 y：绝对坐标
  - dx 和 dy：相对于当前位置的坐标
  - rotate：旋转
  - textLength：字符长度
  - lengthAdjust：控制文本以什么方式伸展到由 textLength 属性定义的长度
  - fill 和 stroke：填充和轮廓也都可以应用于文字
  - CSS 文字属性

- tspan 标签用来标记部分内容，它必须是一个 text 元素的子元素或别的子元素 tspan 的子元素

- textPath：利用它的 xlink:href 属性取得一个任意路径，并且可以让字符顺着路径渲染

#### 渐变

图片和文字、填充和轮廓都可以使用渐变。实现渐变需要借助两个标签，defs 标签用来定义渐变；stop 标签用来定义渐变的颜色坡度，具有三个属性：offset 定义渐变开始和结束的位置、stop-color（定义颜色）和 stop-opacity（定义透明度）

- 线性渐变，其中 x1、y1 是起点，x2、y2 是终点

```js
<linearGradient x1="" y1="" x2="" y2="">
  <stop offset="0%" />
  ...
  <stop offset="20%" />
  ...
  <stop offset="100%" />
</linearGradient>
```

- 径向渐变

```js
<radialGradient cx="" cy="" r="" fx="" fy="">
  <stop offset="0%" />
  ...
  <stop offset="20%" />
  ...
  <stop offset="100%" />
</radialGradient>
```

示例如下

```js
<svg width="500" height="400">
  <defs>
    <radialGradient
      id="radialGradient"
      cx="50%"
      cy="50%"
      r="50%"
      fx="50%"
      fy="50%"
    >
      <stop offset="0%" stop-color="rgb(255, 255, 0)" />
      <stop offset="100%" stop-color="rgb(255, 0, 0)" />
    </radialGradient>
    <radialGradient
      id="radialGradient1"
      cx="50%"
      cy="50%"
      r="50%"
      fx="50%"
      fy="0%"
    >
      <stop offset="0%" stop-color="rgb(255, 255, 0)" />
      <stop offset="100%" stop-color="rgb(255, 0, 0)" />
    </radialGradient>
    <radialGradient
      id="radialGradient2"
      cx="50%"
      cy="50%"
      r="50%"
      fx="0%"
      fy="50%"
    >
      <stop offset="0%" stop-color="rgb(255, 255, 0)" />
      <stop offset="100%" stop-color="rgb(255, 0, 0)" />
    </radialGradient>
  </defs>
  <ellipse cx="100" cy="100" rx="100" ry="50" fill="url(#radialGradient)" />
  <ellipse cx="100" cy="210" rx="100" ry="50" fill="url(#radialGradient1)" />
  <ellipse cx="100" cy="320" rx="100" ry="50" fill="url(#radialGradient2)" />
</svg>
```

#### 裁剪和蒙层

- 裁剪

使用 clipPath 标签定义一条裁剪路径，然后用来裁剪掉元素的部分内容

```js
<svg width="300" height="300">
  <defs>
    <clipPath id="clipPath">
      <path d="M10 50 A50 50 0 0 1 100 50 A50 50 0 0 1 190 50 Q210 100 100 200  Q-5 100 10 50 Z" />
    </clipPath>
  </defs>
  <rect
    x="0"
    y="0"
    width="200"
    height="200"
    fill="#f00"
    clip-path="url(#clipPath)"
  />
</svg>
```

- 蒙层

使用 mask 标签可以显示元素中 mask 标签遮住的内容，区别于 clipPath，可以使用透明度

```js
<svg width="300" height="300">
  <defs>
    <mask id="Mask">
      <path
        d="M10 50 A50 50 0 0 1 100 50 A50 50 0 0 1 190 50 Q210 100 100 200  Q-5 100 10 50 Z"
        fill="#fff"
        fill-opacity="0.5"
      />
    </mask>
  </defs>
  <rect x="0" y="0" width="200" height="200" fill="#f00" mask="url(#Mask)" />
</svg>
```

#### 动画

类似于 css 的 transform 动画

- 平移：transform="translate(x, y)"
- 缩放：transform="scale(x, y)"
- 旋转：transform="rotate(deg)"
- 倾斜：transform="skewX(x) skewY(y)"

SVG 本身不能动态的修改动画内容，需要借助 js 或 css 实现

```js
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>SVG - 动画</title>
  <style>
    svg {
      display: block;
      margin: 50px auto;
    }
    #line {
      stroke-dasharray: 500;
      stroke-dashoffset: 500;
      animation: animation 2s linear infinite;
    }
    @keyframes animation {
      to {
        stroke-dashoffset: 0;
      }
    }
  </style>
</head>
<body>
  <svg width="500" height="500" xmlns="http://www.w3.org/2000/svg" version="1.1">
    <line id="line" x1="0" x2="500" y1="0" y2="0" stroke="orange" stroke-width="10" />
  </svg>
</body>
</html>
```

实际使用中可以借助一些第三方库，如 GreenSock 是一个 svg 动画库
