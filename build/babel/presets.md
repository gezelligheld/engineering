presets 是用来简化 plugins 的使用的，一个 presets 预设了一组 plugin

#### 常用 presets

- @babel/preset-env 所有项目都会用到的
- @babel/preset-flow flow 需要的
- @babel/preset-react react 框架需要的
- @babel/preset-typescript typescript 需要的

示例如下，配置方式同 plugin

```js
module.exports = {
  presets: [
    [
      '@babel/preset-typescript',
      {
        isTSX: true,
        allExtensions: true,
      },
    ],
    [
      '@babel/preset-env',
      {
        modules: false,
        loose: true,
        corejs: 2,
        useBuiltIns: 'entry',
      },
    ],
    '@babel/preset-react',
  ],
  plugins: [],
};
```

#### @babel/preset-env

useBuiltIns 属性决定了添加 Polyfill 策略，默认是 false，即不添加任何的 Polyfill，有以下两种取值

##### entry 模式

根据 targets 配置只引入浏览器不支持的 polyfill，需要在入口文件手动添加一行 import 'core-js'，但无法按需引入

```js
const presets = [
  [
    '@babel/preset-env',
    {
      targets: {
        ios: 8,
        android: 4,
      },
      useBuiltIns: 'entry',
      corejs: 3,
      debug: true, //方便调试
    },
  ],
];
const plugins = [];

module.exports = { presets, plugins };
```

同时需要在业务代码的入口文件下引入 core-js

```js
import 'core-js';
import 'regenerator-runtime/runtime';
```

但是这样引入的 polyfill 太多了，有些可能并不需要，也可以按需引入

```js
import 'core-js/es/promise';
import 'core-js/es/array';
```

##### usage 模式(建议)

根据 targets 配置并检测代码中 ES6 的使用情况按需将 Polyfill 注入，无需在入口文件引入

```js
const presets = [
  [
    '@babel/preset-env',
    {
      targets: {
        ios: 8,
        android: 4.1,
      },
      useBuiltIns: 'usage',
      corejs: 3,
      debug: true,
    },
  ],
];
const plugins = [];

module.exports = { presets, plugins };
```
