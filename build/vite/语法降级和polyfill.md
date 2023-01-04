Vite 语法降级与 Polyfill 注入有一个开箱即用的方案：@vitejs/plugin-legacy，这个插件内部同样使用 @babel/preset-env 以及 core-js 等一系列基础库来进行语法降级和 Polyfill 注入

```js
import legacy from '@vitejs/plugin-legacy';
import { defineConfig } from 'vite';

export default defineConfig({
  plugins: [
    // 省略其它插件
    legacy({
      // 设置目标浏览器，browserslist 配置语法
      targets: ['ie >= 11'],
    }),
  ],
});
```

使用该插件后，Vite 会分别打包出 Modern 模式和 Legacy 模式的产物，然后将两种产物插入同一个 HTML 里面

- Modern 产物被放到 type="module"的 script 标签中，处于高版本浏览器时加载
- Legacy 产物则被放到带有 nomodule 的 script 标签中，处于低版本浏览器时加载

#### 插件执行原理

1. 首先是在 configResolved 钩子中调整了 output 属性，目的是另外打包出一份 Legacy 模式的产物

2. 在 renderChunk 阶段，插件会对 Legacy 模式产物进行语法转译和 Polyfill 收集

3. 进入 generateChunk 钩子阶段，现在 Vite 会对之前收集到的 Polyfill 进行统一的打包

4. 最后将生成的 Legacy 产物及 Polyfill 文件插入到 html 中
