babel 本身不具有任何转化功能，它把转化的功能都分解到一个个 plugin 里面，当不配置任何插件时，经过 babel 的代码和输入是相同的。plugin 会将一些新特性如 for of、let、Set、Map、箭头函数等作 polyfill

#### 分类

##### syntax 语法类

即语法插件，用于 ES 新语法的转换，在解析这一步使 babel 能够解析更多的语法

##### transform 转换类

即转译插件，有以下几个类别

- ES3 对 ES3 的一些特性做转换
- ES5 对 ES5 的一些特性做转换
- ES2015 对 ES6 的特性做转换，大部分 plugin 都是这个类别的
- ES2016 对 ES7 的特性做转换
- ES2017 对 ES8 的特性做转换
- ES2018 对 ES9 的特性做转换
- Modules 自动转换代码的模块组织方式
- Experimental 提案中的特性转换
- Minification 压缩代码体积，这个类别下的插件，没有部署在@babel 里面，而是作为一个独立的库来管理的：babel/minify，而且这个库还是一个实验性的项目，没有发布正式版，所以 babel 没有推荐在生产环境中使用
- React 用于 react 代码转换
- Other 其它

#### 使用

有两种形式

- 纯字符串，用来标识一个 plugin

- 一个数组，这个数组的第一个元素是字符串，用来标识一个 plugin，第二个元素是一个对象字面量，可以往 plugin 传递 options 配置

```js
const plugins = [
  '@babel/plugin-transform-arrow-functions',
  [
    '@babel/plugin-transform-async-to-generator',
    {
      module: 'bluebird',
      method: 'coroutine',
    },
  ],
];
```

> plugin 也可以是本地的一个文件，可以用相对路径或绝对路径引用这个文件，来作为 plugin 的标识

plugin 启用顺序如下

- 配置中 plugins 内直接配置的 plugin，先于 presets 中的 plugin
- 配置中 plugins 数组内的 plugin，按照数组索引顺序启用
- 配置中 presets 数组内的 presets，按照数组索引顺序逆序启用，也就是先应用后面的 presets，再应用前面的 preset

#### plugin 的 options

每个 plugin 的配置不尽相同，几个经常出现的 option 如下

##### targets

指定要兼容的浏览器版本，也可以用 Browserslist 配置语法。也可以在 package.json 中 配置 browserslist，在转码时自动根据我们对转码后代码的目标运行环境的最低版本要求。最佳实践集合如下

```js
// 现代浏览器
last 2 versions and since 2018 and > 0.5%
// 兼容低版本 PC 浏览器
IE >= 11, > 0.5%, not dead
// 兼容低版本移动端浏览器
iOS >= 9, Android >= 4.4, last 2 versions, > 0.2%, not dead
```

##### loose

启用松散式的代码转换，假如某个插件支持这个 option，转换后的代码，会更加简单，代码量更少，但是不会严格遵循 ES 的规格，通常默认是 false

##### spec

启用更加符合 ES 规格的代码转换，默认也是 false，转换后的代码，会增加很多 helper 函数，代码量更大，但是代码质量更好

##### legacy

启用旧的实现来对代码做转换

#### 原理

babel 的运行总共分为三个阶段：解析，转换，生成。首先将 ES6 的代码解析生成 ES6 的 AST，然后将 ES6 的 AST 转换为 ES5 的 AST,最后才将 ES5 的 AST 转化为具体的 ES5 代码

1. 解析

通过 tokenize 函数将源码分割成 tokens，tokens 是一些代码碎片

```js
[
  { type: 'Keyword', value: 'const' },
  { type: 'Identifier', value: 'add' },
  { type: 'Punctuator', value: '=' },
  { type: 'Punctuator', value: '(' },
  { type: 'Identifier', value: 'a' },
  { type: 'Punctuator', value: ',' },
  { type: 'Identifier', value: 'b' },
  { type: 'Punctuator', value: ')' },
  { type: 'Punctuator', value: '=>' },
  { type: 'Identifier', value: 'a' },
  { type: 'Punctuator', value: '+' },
  { type: 'Identifier', value: 'b' },
];
```

[源码](https://github.com/babel/babel/tree/master/packages/babel-parser/src/tokenizer)

将词法分析得到的 tokens 数组转换为 AST 抽象语法树。如一段代码 const add = (a, b) => a + b 的 AST 如下

```js
{
  // 根节点
  "type": "Program",
  "body": [
    {
      "type": "VariableDeclaration", // 变量声明
      "declarations": [ // 具体声明
        {
          "type": "VariableDeclarator", // 变量声明
          "id": {
            "type": "Identifier", // 标识符（最基础的）
            "name": "add" // 函数名
          },
          "init": {
            "type": "ArrowFunctionExpression", // 箭头函数
            "id": null,
            "expression": true,
            "generator": false,
            "params": [ // 参数
              {
                "type": "Identifier",
                "name": "a"
              },
              {
                "type": "Identifier",
                "name": "b"
              }
            ],
            "body": { // 函数体
              "type": "BinaryExpression", // 二项式
              "left": { // 二项式左边
                "type": "Identifier",
                "name": "a"
              },
              "operator": "+", // 二项式运算符
              "right": { // 二项式右边
                "type": "Identifier",
                "name": "b"
              }
            }
          }
        }
      ],
      "kind": "const"
    }
  ],
  "sourceType": "module"
}
```

[源码](https://github.com/babel/babel/tree/master/packages/babel-parser/src/parser)

2. 转换

通过@babel/traverse，对 AST 进行深度优先遍历，增删改含有 type 属性的节点，从而转换成实际需要的 AST

[源码](https://github.com/babel/babel/tree/master/packages/babel-traverse)

3. 代码生成

根据已经修改好的 AST 通过@babel/generator 进行代码生成

[源码](https://github.com/babel/babel/tree/master/packages/babel-generator)

#### 自定义插件

先回顾一下 babel 的处理步骤，首先是语法分析，将字符串形式的代码转换为令牌流，令牌流是一个语法片段的数组，词法分析将其转换为抽象语法树，然后遍历抽象语法树对增删改的操作，也是自定义插件作用的地方，最终深度优先遍历抽象语法树生成代码

如这样一段代码及对应的抽象语法树

```js
function square(n) {
  return n * n;
}
```

```
{
  type: "FunctionDeclaration",
  id: {
    type: "Identifier",
    name: "square"
  },
  params: [{
    type: "Identifier",
    name: "n"
  }],
  body: {
    type: "BlockStatement",
    body: [{
      type: "ReturnStatement",
      argument: {
        type: "BinaryExpression",
        operator: "*",
        left: {
          type: "Identifier",
          name: "n"
        },
        right: {
          type: "Identifier",
          name: "n"
        }
      }
    }]
  }
}
```

##### 访问者 visitor

visitor 对象中可以在抽象语法树中进行节点操作，其中 path 是表示两个节点之间连接的对象，包含节点间的关联关系、操作节点的方法等

```js
module.exports = () => {
  return {
    visitor: {
      Identifier(path) {
        console.log('Called!');
      },
    },
  };
};

// 对于上述示例最终打印四次
```

也可以使用以下方式将一个函数应用到多种类型的节点

```js
module.exports = () => {
  return {
    visitor: {
      'ExportNamedDeclaration|Flow'() {
        console.log('Called!');
      },
    },
  };
};
```

由于是深度优先遍历，当遍历到某个节点的尽头时会有回溯操作，那么节点也就有了向下遍历时的进入和向上回溯时的退出。上面的示例遍历过程如下

```
进入 FunctionDeclaration
    进入 Identifier (id)
    走到尽头
    退出 Identifier (id)
    进入 Identifier (params[0])
    走到尽头
    退出 Identifier (params[0])
    进入 BlockStatement (body)
        进入 ReturnStatement (body)
            进入 BinaryExpression (argument)
                进入 Identifier (left)
                走到尽头
                退出 Identifier (left)
                进入 Identifier (right)
                走到尽头
                退出 Identifier (right)
            退出 BinaryExpression (argument)
        退出 ReturnStatement (body)
    退出 BlockStatement (body)
退出 FunctionDeclaration
```

相应的有两次机会可以访问到节点，如果不声明默认访问的是进入

```js
module.exports = () => {
  return {
    visitor: {
      Identifier: {
        enter() {
          console.log('Entered!');
        },
        exit() {
          console.log('Exited!');
        },
      },
    },
  };
};
```

##### 常用 babel 模块

- babylon：babel 解析器

```js
import * as babylon from 'babylon';

const code = `function square(n) {
  return n * n;
}`;

babylon.parse(code);
// Node {
//   type: "File",
//   start: 0,
//   end: 38,
//   loc: SourceLocation {...},
//   program: Node {...},
//   comments: [],
//   tokens: [...]
// }
```

- babel-traverse：负责替换、移除和添加 ast 节点

```js
import * as babylon from 'babylon';
import traverse from 'babel-traverse';

const code = `function square(n) {
  return n * n;
}`;

const ast = babylon.parse(code);

traverse(ast, {
  enter(path) {
    if (path.node.type === 'Identifier' && path.node.name === 'n') {
      path.node.name = 'x';
    }
  },
});
```

- babel-types：包含构造、验证以及变换 AST 节点的方法的工具库

```js
import traverse from 'babel-traverse';
import * as t from 'babel-types';

traverse(ast, {
  enter(path) {
    if (t.isIdentifier(path.node, { name: 'n' })) {
      path.node.name = 'x';
    }
  },
});
```

- babel-generator：代码生成器，读取 AST 并将其转换为代码和源码映射

```js
import * as babylon from 'babylon';
import generate from 'babel-generator';

const code = `function square(n) {
  return n * n;
}`;

const ast = babylon.parse(code);

generate(ast, {}, code);
// {
//   code: "...",
//   map: "..."
// }
```

- babel-template：让你编写字符串形式且带有占位符的代码来代替手动编码

```js
import template from 'babel-template';
import generate from 'babel-generator';
import * as t from 'babel-types';

const buildRequire = template(`
  var IMPORT_NAME = require(SOURCE);
`);

const ast = buildRequire({
  IMPORT_NAME: t.identifier('myModule'),
  SOURCE: t.stringLiteral('my-module'),
});

console.log(generate(ast).code);
```

##### 编写一个插件

可以直接在 babel plugin 配置传递一个自定义 plugin 的路径

```
{
  plugins: [
    ["./myplugin.js", {
      "option1": true,
      "option2": false
    }]
  ]
}
```

插件的 option 通过 state 对象传递

```js
export default function ({ types: t }) {
  return {
    visitor: {
      FunctionDeclaration(path, state) {
        console.log(state.opts); // { option1: true, option2: false }
      },
    },
  };
}
```

在插件之前或之后有钩子可以做预备或清除的动作

```js
export default function ({ types: t }) {
  return {
    pre(state) {
      this.cache = new Map();
    },
    visitor: {
      StringLiteral(path) {
        this.cache.set(path.node.value, 1);
      },
    },
    post(state) {
      console.log(this.cache);
    },
  };
}
```

参考

1. [babel 插件手册官方文档](https://github.com/jamiebuilds/babel-handbook/blob/master/translations/zh-Hans/plugin-handbook.md)
2. [babel 插件示例](https://github.com/QuarkGluonPlasma/babel-plugin-exercize)
