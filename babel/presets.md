presets是用来简化plugins的使用的，一个presets预设了一组plugin

#### 常用presets

- @babel/preset-env 所有项目都会用到的

- @babel/preset-flow flow需要的

- @babel/preset-react react框架需要的

- @babel/preset-typescript typescript需要的

示例如下，配置方式同plugin

```js
module.exports = {
    presets: [
        ['@babel/preset-typescript', {
            isTSX: true,
            allExtensions: true
        }],
        ['@babel/preset-env', {
            modules: false,
            loose: true,
            corejs: 2,
            useBuiltIns: 'entry'
        }],
        '@babel/preset-react'
    ],
    plugins: []
}
```

#### @babel/preset-env

- browserslist

根据package.json中browserslist的配置，在转码时自动根据我们对转码后代码的目标运行环境的最低版本要求

- entry模式

在useBuiltIns: 'entry'模式下，代码中对core-js的import：import "core-js"，会根据targets的配置，替换为core-js最底层的modules引用, .babelrc.js如下配置

```js
const presets = [ 
    [
    "@babel/preset-env",
            {
                "targets": {
                    ios: 8,
                    android: 4
                },
                useBuiltIns: "entry",
                corejs: 3,
                debug: true //方便调试
            }
        ]
];
const plugins = [
];

module.exports = { presets, plugins };
```

同时需要在业务代码的入口文件下引入core-js

```js
import "core-js";
import "regenerator-runtime/runtime";
```

但是这样引入的polyfill太多了，有些可能并不需要，也可以按需引入

```js
import "core-js/es/promise";
import "core-js/es/array";
```

- usage模式(建议)

usage比起entry，最大的好处就是，它会根据每个文件里面，用到了哪些es的新特性，然后根据我们设置的targets判断，是否需要polyfill，如果targets的最低环境不支持某个es特性，则这个es特性的core-js的对应module会被注入

```js
const presets = [ 
    [
    "@babel/preset-env",
            {
                "targets": {
                    ios: 8,
                    android: 4.1
                },
                useBuiltIns: "usage",
                corejs: 3,
                debug: true
            }
        ]
];
const plugins = [
];

module.exports = { presets, plugins };
```