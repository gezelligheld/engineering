#### @babel/polyfill

等价于

```js
import "core-js/stable";
import "regenerator-runtime/runtime";
```

现已废弃，他是通过间接引用core-js的模块来实现的，会无差别地引入所有的core-js的stable状态的特性，这是不符合未来浏览器等运行环境的

现有的方式是，@babel/preset-env提供了useBuiltIns: "entry"和useBuiltIns: "usage"两种polyfill的方式，这两种方式可根据开发者配置的browserslist的目标运行环境，自动引入core-js最小化的polyfill modules组合，更加符合实际需求

#### runtime

- @babel/plugin-transform-runtime

开发环境下的依赖

- @babel/runtime

生产环境下的依赖，babel的内部库

他们主要有两方面的用途：

- babel在转码过程中，会加入很多babel自己的helper函数，这些helper函数，在每个文件里可能都会重复存在，transform-runtime插件可以把这些重复的helper函数，转换成公共的、单独的依赖引入，从而节省转码后的文件大小

- 开发者在代码中如果使用了新的ES特性，往往需要通过core-js和regenerator-runtime给全局环境注入polyfill。 这种做法在应用型的开发中，是非常标准的做法。 但是如果在开发一个独立的工具库项目，不确定它将会被其它人用到什么运行环境里面，那么前面那种扩展全局环境的polyfill就不是一个很好的方式。 transform-runtime可以帮助这种项目创建一个沙盒环境，即使在代码里用到了新的ES特性，它能将这些特性对应的全局变量，转换为对core-js和regenerator-runtime非全局变量版本的引用

#### 对regenerator-runtime的polyfill

```js
// babel.config.js
const presets = [ 
    [
        "@babel/preset-env",
        {
            useBuiltIns: false
        }
    ]
];
const plugins = [
];

module.exports = { presets, plugins };

// 要转换的代码
export default async function () {
    await 'hi';
}

// 结果
"use strict";

Object.defineProperty(exports, "__esModule", {
  value: true
});
exports.default = _default;

function asyncGeneratorStep(gen, resolve, reject, _next, _throw, key, arg) { try { var info = gen[key](arg); var value = info.value; } catch (error) { reject(error); return; } if (info.done) { resolve(value); } else { Promise.resolve(value).then(_next, _throw); } }

function _asyncToGenerator(fn) { return function () { var self = this, args = arguments; return new Promise(function (resolve, reject) { var gen = fn.apply(self, args); function _next(value) { asyncGeneratorStep(gen, resolve, reject, _next, _throw, "next", value); } function _throw(err) { asyncGeneratorStep(gen, resolve, reject, _next, _throw, "throw", err); } _next(undefined); }); }; }

function _default() {
  return _ref.apply(this, arguments);
}

function _ref() {
  _ref = _asyncToGenerator(
  /*#__PURE__*/
  regeneratorRuntime.mark(function _callee() {
    return regeneratorRuntime.wrap(function _callee$(_context) {
      while (1) {
        switch (_context.prev = _context.next) {
          case 0:
            _context.next = 2;
            return 'hi';

          case 2:
          case "end":
            return _context.stop();
        }
      }
    }, _callee);
  }));
  return _ref.apply(this, arguments);
}
```

regeneratorRuntime是一个全局变量，如果其它人引用你这段代码，同时又不知道要在运行环境里面添加regenerator-runtime的polyfill，这段代码别人引用过去运行就会报错。 transform-runtime可以帮助开发者，解决这个问题

```js
//babel.config.js
const presets = [ 
    [
        "@babel/preset-env",
        {
            useBuiltIns: false
        }
    ]
];
const plugins = [
    [
        "@babel/plugin-transform-runtime"
    ]
];

module.exports = { presets, plugins };

// 结果
"use strict";

var _interopRequireDefault = require("@babel/runtime/helpers/interopRequireDefault");

Object.defineProperty(exports, "__esModule", {
  value: true
});
exports.default = _default;

var _regenerator = _interopRequireDefault(require("@babel/runtime/regenerator"));

var _asyncToGenerator2 = _interopRequireDefault(require("@babel/runtime/helpers/asyncToGenerator"));

function _default() {
  return _ref.apply(this, arguments);
}

function _ref() {
  _ref = (0, _asyncToGenerator2.default)(
  /*#__PURE__*/
  _regenerator.default.mark(function _callee() {
    return _regenerator.default.wrap(function _callee$(_context) {
      while (1) {
        switch (_context.prev = _context.next) {
          case 0:
            _context.next = 2;
            return 'hi';

          case 2:
          case "end":
            return _context.stop();
        }
      }
    }, _callee);
  }));
  return _ref.apply(this, arguments);
}
```

transform-runtime通过引用@babel/runtime/regenerator和@babel/runtime/helpers/asyncToGenerator这两个内部模块，消除了regeneratorRuntime这个全局变量

对core-js相关polyfill的处理也是类似的

需要注意的是不能同时开启preset-env和transform-runtime的polyfill，transform-runtime的polyfill，对目标环境是不做判断的，只要它识别到代码里有用到新的ES特性，就会进行替换；而preset-env的polyfill会根据目标环境进行替换，两种polyfill应用于不同的场景