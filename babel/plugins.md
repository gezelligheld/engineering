babel 本身不具有任何转化功能，它把转化的功能都分解到一个个 plugin 里面，当不配置任何插件时，经过 babel 的代码和输入是相同的。plugin会将一些新特性如for of、let、Set、Map、箭头函数等作polyfill

明确一点，babel 的运行总共分为三个阶段：解析，转换，生成

#### 分类

##### syntax 语法类

即语法插件，用于ES新语法的转换，在解析这一步使 babel 能够解析更多的语法

##### transform 转换类

即转译插件，有以下几个类别

- ES3 对ES3的一些特性做转换

- ES5 对ES5的一些特性做转换

- ES2015 对ES6的特性做转换，大部分plugin都是这个类别的

- ES2016 对ES7的特性做转换

- ES2017 对ES8的特性做转换

- ES2018 对ES9的特性做转换

- Modules 自动转换代码的模块组织方式

- Experimental 提案中的特性转换

- Minification 压缩代码体积，这个类别下的插件，没有部署在@babel里面，而是作为一个独立的库来管理的：babel/minify，而且这个库还是一个实验性的项目，没有发布正式版，所以babel没有推荐在生产环境中使用

- React 用于react代码转换

- Other 其它

#### 使用

有两种形式

- 纯字符串，用来标识一个plugin

- 一个数组，这个数组的第一个元素是字符串，用来标识一个plugin，第二个元素是一个对象字面量，可以往plugin传递options配置

```js
const plugins = [
    '@babel/plugin-transform-arrow-functions',
    [
      "@babel/plugin-transform-async-to-generator",
      {
         "module": "bluebird",
         "method": "coroutine"
       }
    ]
];
```
> plugin也可以是本地的一个文件，可以用相对路径或绝对路径引用这个文件，来作为plugin的标识

#### 启用顺序

- 配置中plugins内直接配置的plugin，先于presets中的plugin

- 配置中plugins数组内的plugin，按照数组索引顺序启用

- 配置中presets数组内的presets，按照数组索引顺序逆序启用，也就是先应用后面的presets，再应用前面的preset

#### plugin的options

每个plugin的配置不尽相同，几个经常出现的option如下

- loose

启用松散式的代码转换，假如某个插件支持这个option，转换后的代码，会更加简单，代码量更少，但是不会严格遵循ES的规格，通常默认是false

- spec

启用更加符合ES规格的代码转换，默认也是false，转换后的代码，会增加很多helper函数，代码量更大，但是代码质量更好

- legacy

启用旧的实现来对代码做转换

- useBuiltIns

如果为true，则在转换过程中，会尽可能地使用运行环境已经支持的实现，而不是引入polyfill

#### 原理

babel 的工作原理就是将 ES6 的代码解析生成ES6的AST，然后将 ES6 的 AST 转换为 ES5 的AST,最后才将 ES5 的 AST 转化为具体的 ES5 代码

1. 解析

1.1 词法分析

tokens -> ast -> 转换ast -> 代码生成

通过tokenize函数将源码分割成tokens，tokens是一些代码碎片

```js
[
    { "type": "Keyword", "value": "const" },
    { "type": "Identifier", "value": "add" },
    { "type": "Punctuator", "value": "=" },
    { "type": "Punctuator", "value": "(" },
    { "type": "Identifier", "value": "a" },
    { "type": "Punctuator", "value": "," },
    { "type": "Identifier", "value": "b" },
    { "type": "Punctuator", "value": ")" },
    { "type": "Punctuator", "value": "=>" },
    { "type": "Identifier", "value": "a" },
    { "type": "Punctuator", "value": "+" },
    { "type": "Identifier", "value": "b" }
]
```

[源码](https://github.com/babel/babel/tree/master/packages/babel-parser/src/tokenizer)

1.2 语法分析

将词法分析得到的tokens数组转换为AST抽象语法树

如一段代码const add = (a, b) => a + b的AST如下

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

通过@babel/traverse，对AST进行深度优先遍历，增删改含有type属性的节点，从而转换成实际需要的AST

[源码](https://github.com/babel/babel/tree/master/packages/babel-traverse)

3. 代码生成

根据已经修改好的AST通过@babel/generator进行代码生成

[源码](https://github.com/babel/babel/tree/master/packages/babel-generator)