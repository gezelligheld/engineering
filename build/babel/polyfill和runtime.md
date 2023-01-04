#### @babel/polyfill

babel 默认只转换 js 语法，而不转换新的 API，比如 Iterator、Generator、Set、Maps、Proxy、Reflect、Symbol、Promise 等全局对象，以及一些定义在全局对象上的方法，如 Object.assign，这些都不会转码，如果想在低版本的环境中运行这个方法，就需要安装@babel/polyfill

@babel/polyfill 内部集成了 core-js 和 regenerator，等价于

```js
import 'core-js/stable';
import 'regenerator-runtime/runtime';
```

现已废弃，他是通过间接引用 core-js 的模块来实现的，会无差别地引入所有的 core-js 的 stable 状态的特性，这是不符合未来浏览器等运行环境的

更好的方式是，@babel/preset-env 提供了 entry 和 usage 两种 polyfill 的方式，这两种方式可根据开发者配置的 browserslist 的目标运行环境，自动引入 core-js 最小化的 polyfill modules 组合，更加符合实际需求，但是存在一些问题

- 如果使用新特性，往往是通过基础库(如 core-js)往全局环境添加 Polyfill，如果是开发应用没有任何问题，如果是开发第三方工具库，则很可能会对全局空间造成污染
- 很多工具函数的实现代码，会在许多文件中重现出现，造成文件体积冗余

#### 更好的 Polyfill 方案：transform-runtime

transform-runtime 方案可以作为@babel/preset-env 中 useBuiltIns 配置的替代品，使用 transform-runtime 方案应该将 useBuiltIns 设为 false。相关插件有两个

- @babel/plugin-transform-runtime：编译时工具，用来转换语法和添加 Polyfill

- @babel/runtime：运行时基础库，包括以下内容

  - core-js：转换一些内置类 (Promise, Symbols 等等) 和静态方法 (Array.from 等)
  - regenerator：core-js 的拾遗补漏，主要是 generator/yield 和 async/await 两组的支持
  - helpers：各种语法转换用到的工具函数

```json
{
  "plugins": [
    // 添加 transform-runtime 插件
    [
      "@babel/plugin-transform-runtime",
      {
        "corejs": 3
      }
    ]
  ],
  "presets": [
    [
      "@babel/preset-env",
      {
        "targets": {
          "ie": "11"
        },
        "corejs": 3,
        // 关闭 @babel/preset-env 默认的 Polyfill 注入
        "useBuiltIns": false,
        "modules": false
      }
    ]
  ]
}
```

他们主要有两方面的用途：

- babel 在转码过程中，会加入很多 babel 自己的 helper 函数，这些 helper 函数，在每个文件里可能都会重复存在，transform-runtime 插件可以把这些重复的 helper 函数，转换成公共的、单独的依赖引入，从而节省转码后的文件大小

- transform-runtime 可以帮助这种项目创建一个沙盒环境，即使在代码里用到了新的 ES 特性，它能将这些特性对应的全局变量，转换为对 core-js 和 regenerator-runtime 非全局变量版本的引用，避免了全局空间的污染

例如，以 async/await 为例

```js
function _asyncToGenerator(fn) { return function () {....}} // 很长很长一段

var _ref = _asyncToGenerator(function* (arg1, arg2) {
  yield (0, something)(arg1, arg2);
});
```

引入@babel/plugin-transform-runtime 后，由重复定义变成了重复引入

```js
var _asyncToGenerator2 = require('babel-runtime/helpers/asyncToGenerator');
var _asyncToGenerator3 = _interopRequireDefault(_asyncToGenerator2);

var _ref = _asyncToGenerator3(function* (arg1, arg2) {
  yield (0, something)(arg1, arg2);
});
```
