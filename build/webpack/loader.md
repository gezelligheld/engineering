#### 概念

webpack 的编译器，它使得 webpack 可以处理一些非 JavaScript 文件，比如 png、csv、xml、css、json 等，使用合适的 loader 可以让 JavaScript 的 import 导入非 JavaScript 模块，最终打包编译为标准的 js 代码

> loader 的执行顺序和配置顺序是相反的，配置的最后一个 loader 先执行

#### 常用 loader

##### 处理样式

- css-loader

处理@import 或 url 引入的外部资源

- style-loader

创建 style 标签引入样式

```js
const config = {
  // ...
  module: {
    rules: [
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader'],
      },
    ],
  },
};
```

- postcss-loader

扩展 css 语法，可以使用 autoprefixer（自动添加不同浏览器下的样式前缀）等插件

```js
const config = {
  // ...
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          // 用于抽离css样式为link标签引入
          MiniCssExtractPlugin.loader,
          'css-loader',
          'postcss-loader',
          'resolve-url-loader',
          {
            loader: 'sass-loader',
            options: {
              sourceMap: true,
            },
          },
        ],
      },
    ],
  },
};

// postcss.config.js
const autoprefixer = require('autoprefixer');

module.exports = {
  plugins: [
    autoprefixer({
      // 目前只有发包的构建使用到postcss,所以直接将env设置为production
      env: 'production',
    }),
  ],
};
```

##### 处理文件

- url-loader

加载文件，可以设置阈值，小于阈值时把文件转为 base64 编码，这样无需再从服务器拉取图片

```js
const config = {
  // ...
  module: {
    rules: [
      {
        test: /\.(png|gif|jpe?g|svg)$/,
        use: [
          {
            loader: require.resolve('url-loader'),
            options: {
              limit: 10000,
              name: 'img/[name].[ext]',
            },
          },
        ],
      },
    ],
  },
};
```

- file-loader：用来指定资源的访问路径和输出路径

```js
{
    loader: require.resolve('file-loader'),
    exclude: [/\.(js|mjs|jsx|ts|tsx)$/, /\.html$/, /\.json$/],
    options: {
        name: 'static/media/[name].[hash:8].[ext]',
    },
}
```

- @svgr/webpack：可以将 svg 直接导入，不做转换

##### 处理编译

- babel-loader

es6+转为 es5

```js
const config = {
  // ...
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        use: [
          {
            loader: 'babel-loader',
            options: {
              // babel配置项
              ...babelOptions,
            },
          },
        ],
      },
    ],
  },
};
```

- ts-loader：ts 转为 js

##### 校验测试

- eslint-loader

代码检查

```js
const config = {
  module: {
    rules: [
      {
        enforce: 'pre',
        // 意思是，当遇到导入语句中含有jsx或tsx的路径时，对其打包前需要用eslint-loader转换以下
        test: /\.(jsx?|tsx?)$/,
        exclude: /(node_modules|dist)/,
        use: {
          loader: require.resolve('eslint-loader'),
          options: {
            eslintPath: require.resolve('eslint'),
            configFile: path.join(__dirname, '../eslint/eslintrc.dev.js'),
          },
        },
      },
    ],
  },
};
```

##### 性能优化

- cache-loader

对一些性能开销比较大的 loader 前添加 cache-loader，将其结果缓存到磁盘

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.ext$/,
        use: ['cache-loader', ...loaders],
        include: path.resolve('src'),
      },
    ],
  },
};
```

#### 自定义 loader

自定义 loader 是一个接收对应类型文件代码参数、返回转换后的代码的函数，示例如下

```js
// reverse-loader.js
module.exports = function (src) {
  if (src) {
    console.log('--- reverse-loader input:', src);
    src = src.split('').reverse().join('');
    console.log('--- reverse-loader output:', src);
  }
  return src;
};
// uppercase-loader.js
module.exports = function (src) {
  if (src) {
    console.log('--- uppercase-loader input:', src);
    src = src.charAt(0).toUpperCase() + src.slice(1);
    console.log('--- uppercase-loader output:', src);
  }
  return `module.exports = '${src}'`;
};
// webpack.config.js
module.exports = {
  module: {
    rules: [
      ...,
      {
        test: /\.txt$/,
        use: [
          './loader/uppercase-loader.js',
          './loader/reverse-loader.js'
        ]
      }
    ]
  }
}
```
