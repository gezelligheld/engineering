plugin会将一些新特性如for of、let、Set、Map、箭头函数等作polyfillß

#### 分类

babel的plugin分为三类

##### syntax 语法类

syntax类plugin用于ES新语法的转换，会被transform类或proposal类依赖，用于语法解析

##### transform 转换类

有以下几个类别

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

##### proposal 也是转换类，指代那些对ES Proposal进行转换的plugin

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