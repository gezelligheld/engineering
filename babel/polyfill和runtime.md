#### @babel/polyfill

babel 默认只转换 js 语法，而不转换新的 API，比如 Iterator、Generator、Set、Maps、Proxy、Reflect、Symbol、Promise 等全局对象，以及一些定义在全局对象上的方法，如Object.assign，这些都不会转码，如果想在低版本的环境中运行这个方法，就需要安装@babel/polyfill

@babel/polyfill内部集成了core-js 和 regenerator，等价于

```js
import "core-js/stable";
import "regenerator-runtime/runtime";
```

现已废弃，他是通过间接引用core-js的模块来实现的，会无差别地引入所有的core-js的stable状态的特性，这是不符合未来浏览器等运行环境的

现有的方式是，@babel/preset-env提供了useBuiltIns: "entry"和useBuiltIns: "usage"两种polyfill的方式，这两种方式可根据开发者配置的browserslist的目标运行环境，自动引入core-js最小化的polyfill modules组合，更加符合实际需求

#### runtime

- @babel/plugin-transform-runtime

开发环境下的依赖，依赖@babel/runtime

- @babel/runtime

生产环境下的依赖，babel的内部库，内部集成了以下内容

  - core-js：转换一些内置类 (Promise, Symbols等等) 和静态方法 (Array.from 等)

  - regenerator：core-js 的拾遗补漏，主要是 generator/yield 和 async/await 两组的支持

  - helpers：内部的辅助函数

他们主要有两方面的用途：

- babel在转码过程中，会加入很多babel自己的helper函数，这些helper函数，在每个文件里可能都会重复存在，transform-runtime插件可以把这些重复的helper函数，转换成公共的、单独的依赖引入，从而节省转码后的文件大小

- 开发者在代码中如果使用了新的ES特性，往往需要通过core-js和regenerator-runtime给全局环境注入polyfill。 这种做法在应用型的开发中，是非常标准的做法。 但是如果在开发一个独立的工具库项目，不确定它将会被其它人用到什么运行环境里面，那么前面那种扩展全局环境的polyfill就不是一个很好的方式。 transform-runtime可以帮助这种项目创建一个沙盒环境，即使在代码里用到了新的ES特性，它能将这些特性对应的全局变量，转换为对core-js和regenerator-runtime非全局变量版本的引用

例如，以async/await为例

```js
function _asyncToGenerator(fn) { return function () {....}} // 很长很长一段

var _ref = _asyncToGenerator(function* (arg1, arg2) {
  yield (0, something)(arg1, arg2);
});
```

引入@babel/plugin-transform-runtime后，由重复定义变成了重复引入

```js
var _asyncToGenerator2 = require('babel-runtime/helpers/asyncToGenerator');
var _asyncToGenerator3 = _interopRequireDefault(_asyncToGenerator2);

var _ref = _asyncToGenerator3(function* (arg1, arg2) {
  yield (0, something)(arg1, arg2);
});
```