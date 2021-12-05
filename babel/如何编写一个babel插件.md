先回顾一下babel的处理步骤，首先是语法分析，将字符串形式的代码转换为令牌流，令牌流是一个语法片段的数组，词法分析将其转换为抽象语法树，然后遍历抽象语法树对增删改的操作，也是自定义插件作用的地方，最终深度优先遍历抽象语法树生成代码

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

#### 访问者visitor

visitor对象中可以在抽象语法树中进行节点操作，其中 path 是表示两个节点之间连接的对象，包含节点间的关联关系、操作节点的方法等

```js
module.exports = () => {
    return {
        visitor: {
            Identifier(path) {
                console.log("Called!");
            }
        }
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
                console.log("Called!");
            }
        }
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
                    console.log("Entered!");
                },
                exit() {
                    console.log("Exited!");
                }
            }
        }
    };
};
```

#### 常用babel模块

- babylon

babel解析器

```js
import * as babylon from "babylon";

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

- babel-traverse

负责替换、移除和添加ast节点

```js
import * as babylon from "babylon";
import traverse from "babel-traverse";

const code = `function square(n) {
  return n * n;
}`;

const ast = babylon.parse(code);

traverse(ast, {
  enter(path) {
    if (
      path.node.type === "Identifier" &&
      path.node.name === "n"
    ) {
      path.node.name = "x";
    }
  }
});
```

- babel-types

包含构造、验证以及变换 AST 节点的方法的工具库

```js
import traverse from "babel-traverse";
import * as t from "babel-types";

traverse(ast, {
  enter(path) {
    if (t.isIdentifier(path.node, { name: "n" })) {
      path.node.name = "x";
    }
  }
});
```

- babel-generator

代码生成器，读取AST并将其转换为代码和源码映射

```js
import * as babylon from "babylon";
import generate from "babel-generator";

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

- babel-template

让你编写字符串形式且带有占位符的代码来代替手动编码

```js
import template from "babel-template";
import generate from "babel-generator";
import * as t from "babel-types";

const buildRequire = template(`
  var IMPORT_NAME = require(SOURCE);
`);

const ast = buildRequire({
  IMPORT_NAME: t.identifier("myModule"),
  SOURCE: t.stringLiteral("my-module")
});

console.log(generate(ast).code);
```

#### 编写一个插件

可以直接在babel plugin配置传递一个自定义plugin的路径

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

插件的option通过state对象传递

```js
export default function({ types: t }) {
  return {
    visitor: {
      FunctionDeclaration(path, state) {
        console.log(state.opts); // { option1: true, option2: false }
      }
    }
  }
}
```

在插件之前或之后有钩子可以做预备或清除的动作

```js
export default function({ types: t }) {
  return {
    pre(state) {
      this.cache = new Map();
    },
    visitor: {
      StringLiteral(path) {
        this.cache.set(path.node.value, 1);
      }
    },
    post(state) {
      console.log(this.cache);
    }
  };
}
```

参考
1. (babel插件手册官方文档)[https://github.com/jamiebuilds/babel-handbook/blob/master/translations/zh-Hans/plugin-handbook.md]