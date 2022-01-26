tree shaking用于移除js上下文未引用的代码，以减少产出包大小

#### 原理

动态导入是无法tree shaking的，因为不能确定实际运行之前那些模块是不需要加载的

```js
let m;

if (condition) {
    // commonjs规范
    m = require('./a');
}
else {
    m = require('./b');
}
```

es6添加了静态导入，整个依赖树可以静态的导出静态语法树，很容易地去tree shaking，因为在代码不运行的情况就能得知哪些模块是没有被引入的

```js
import a from './a';
import b from './b';

let m;

if (condition) {
    m = a;
}
else {
    m = b;
}
```

#### 使用

webpack设置mode为production时会自动开启tree shaking

##### side effects

side effects是指import某些内容时执行的一些动作，但并没有export。如导入了poloyfill

```js
import 'abortcontroller-polyfill/dist/polyfill-patch-fetch';
```

tree shaking并不能识别到哪些是side effects，可以通过package.json来指定

```js
{
    "name": "xxx",
    "sideEffects": [
        "./src/common/a.js"
    ]
}
```