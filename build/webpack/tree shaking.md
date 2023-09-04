tree shaking 用于移除 js 上下文未引用的代码，以减少产出包大小，依赖 esm，不局限于 webpack

#### 原理

动态导入是无法 tree shaking 的，因为不能确定实际运行之前那些模块是不需要加载的

```js
let m;

if (condition) {
  // commonjs规范
  m = require('./a');
} else {
  m = require('./b');
}
```

es6 添加了静态导入，整个依赖树可以静态的导出静态语法树，在代码不运行的情况就能得知哪些模块是没有被引入的。Tree-shaking 算法首先标记所有相关语句，然后删除无效代码，其思想类似于标记清除算法

```js
import a from './a';
import b from './b';

let m;

if (condition) {
  m = a;
} else {
  m = b;
}
```

#### 使用

webpack 设置 mode 为 production 时会自动开启 tree shaking

##### side effects

side effects 是指 import 某些内容时执行的一些动作，但并没有 export。如导入了 poloyfill

```js
import 'abortcontroller-polyfill/dist/polyfill-patch-fetch';
```

tree shaking 并不能识别到哪些是 side effects，可以通过 package.json 来指定

```js
{
    "name": "xxx",
    "sideEffects": [
        "./src/common/a.js"
    ]
}
```
