window.requestAnimationFrame() 告诉浏览器你希望执行一个动画，并且要求浏览器在下次重绘之前调用指定的回调函数更新动画。回调函数执行次数通常是每秒 60 次，每执行一次大约是 16.6ms

> 为了提高性能和电池寿命，因此在大多数浏览器里，当 requestAnimationFrame() 运行在后台标签页或者隐藏的 iframe 里时会被暂停调用

```js
const element = document.getElementById('some-element-you-want-to-animate');
let start;

function step(timestamp) {
  if (start === undefined) start = timestamp;
  const elapsed = timestamp - start;
  element.style.transform =
    'translateX(' + Math.min(0.1 * elapsed, 200) + 'px)';

  if (elapsed < 2000) {
    // 在两秒后停止动画
    window.cancelAnimationFrame(step);
  }
}

window.requestAnimationFrame(step);
```

#### 浏览器每一帧做的事情

requestAnimationFrame 的回调在一帧中的执行时机是，Chrome、Firefox 等符合标准的浏览器会在 style/layout/paint 之前触发，而 IE 等浏览器中则是在 style/layout/paint 之后触发

![](https://user-images.githubusercontent.com/1249423/42104778-74e0cea2-7c00-11e8-8877-0e3713f4a431.png)

流程如下

1. 如果当前浏览上下文中没有到渲染时机则取消渲染，渲染时机根据硬件刷新率限制、页面性能或页面是否在后台等因素

2. Vsync 事件触发后, 一个帧开始了，Vsync 事件是垂直同步技术的内容，用来同步屏幕刷新率

3. 所有输入事件处理程序（touchmove，scroll，click）应首先触发，每帧一次，但不一定是这样，调度程序会尽力尝试

4. 解析 html，任何新添加的 HTML 都需要被处理，并创建 DOM 元素

5. 计算样式，任何新添加或修改的内容都需要重新计算样式

6. 为每个可见元素计算几何信息，即大小和位置（layout）

7. 创建堆叠上下文和深度排序元素的过程（layer）

8. 然后绘制任何新建的或者视觉发生变化的元素

9. 计算图层和图块信息并将其传递回合成线程处理，进行栅格化

10. 随着各个图层的图块都被栅格化、任何新图块都将和输入数据一起提交给 GPU 线程

11. GPU 线程将图块上传到 GPU 绘制到屏幕上

12. 如果主线程在一帧结束时有空闲时间，则会触发 requestIdleCallback 回调
