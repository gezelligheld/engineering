CSS Modules是所有的类名和动画名称默认都有各自的作用域的CSS文件,在构建时对对CSS类名和选择器限定作用域的一种方式

构建过程中,构建工具会将把以一定规则命名的样式文件(如index.module.css)的类名编译成一个独一无二的哈希值,可以实现组件间的样式隔离,不会影响其他模块的样式

#### 配置和使用

webpack配置css modules如下

```js
// webpack.config.js
module.exports = {
    // ...
    module: {
        rules: [
            {
                test: /\.module\.css$/,
                loader: "style-loader!css-loader?modules"
            }
        ]
    }
}
```

使用如下

```js
// index.css
.content {
    width: 100px;
    height: 100px;
}

// index.js
import style from './index.css';
export default () => (
    <div className={style.content}></div>
);
```

#### 全局作用域

默认均为局部作用域,需要特殊的语法声明为全局作用域

```css
:global(.title) {
  color: green;
}
```

#### 修改类名哈希值命名规则

```js
module.exports = {
    // ...
    module: {
        rules: [
            {
                test: /\.module\.css$/,
                loader: "style-loader!css-loader?modules&localIdentName=[path][name]---[local]---[hash:base64:5]"
            }
        ]
    }
}
```

参考文档:
1. [CSS Modules入门Ⅰ：它是什么？为什么要使用它？](https://zhuanlan.zhihu.com/p/23571898)