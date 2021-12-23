例如，phone6 的屏幕宽度为 375px，设计师做的视觉稿一般是 750px，也就是 2x 图，这个时候设计师在设计稿上画了 1px 的边框，如果你在 css 中也写了border-width: 1px，那么问题就出现了

iphone6 的设备像素比是 2，这里的物理像素是 1px，此时的设备独立像素为0.5px

```
设备独立像素（css 像素）= 物理像素 / 设备像素比 = 0.5px
```

但是css不支持0.5px，于是有了各种hack方法，对比如下

![](https://mengsixing.github.io/blog/css-devicePixelRatio.png)

- 使用缩放

```html
<style>
  .hr.scale-half {
    height: 1px;
    transform: scaleY(0.5);
    /* 设置旋转元素的基点位置，不加会虚化 */
    transform-origin: 50% 100%;
  }
</style>
<div class="hr scale-half"></div>
```

- 使用线性渐变

从上向下从白色渐变到黑色，1px的设备独立像素是2px的物理像素，所以第一个1px是白色，第二个是黑色，到达显示一半的效果

实际效果一般，会虚

```html
<style>
  .hr.gradient {
    height: 1px;
    background: linear-gradient(0deg, #fff, #000);
  }
</style>
<div class="hr gradient"></div>
```

- 使用 boxshadow

但是 safari 不支持小于 1px 的 boxshadow

```html
<style>
  .hr.boxshadow {
    height: 1px;
    background: none;
    box-shadow: 0 0.5px 0 #000;
  }
</style>
<div class="hr boxshadow"></div>
```

- 使用 svg

使用 svg 的 line 元素画线，stroke 表示描边颜色，默认描边宽度 stroke-width="1"，由于 svg 的描边等属性的 1px 就是物理像素的 1px，所以就相当于高清屏的 0.5px

```html
<style>
  .hr.svg {
    /* 直接使用svg */
    background: url("data:image/svg+xml;utf-8,<svg xmlns='http://www.w3.org/2000/svg' width='100%' height='1px'><line x1='0' y1='0' x2='100%' y2='0' stroke='#000'></line></svg>");
    /* 使用 base64 格式的svg，解决 firefox 兼容性问题 */
    background: url("data:image/svg+xml;base64,PHN2ZyB4bWxucz0naHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmcnIHdpZHRoPScxMDAlJyBoZWlnaHQ9JzFweCc+PGxpbmUgeDE9JzAnIHkxPScwJyB4Mj0nMTAwJScgeTI9JzAnIHN0cm9rZT0nIzAwMCc+PC9saW5lPjwvc3ZnPg==");
  }
</style>
<div class="hr svg"></div>
```

- 使用 viewport

默认的缩放比例为 1，如果知道设备像素比，就能计算出需要缩放的比例，实现 0.5px 的效果了

通过 js 获取 devicePixelRatio，然后动态修改 viewport 中的缩放比例

```html
<meta name="viewport" content="initial-scale=0.5, user-scalable=no" />
```