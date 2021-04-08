#### 概念

webpack的编译器，它使得webpack可以处理一些非JavaScript文件，比如png、csv、xml、css、json等，使用合适的loader可以让JavaScript的import导入非JavaScript模块

> loader的执行顺序和配置顺序是相反的，配置的最后一个loader先执行

#### 常用loader

- file-loader

加载文件

- url-loader

```js
const config = {
    // ...
    module: {
        rules: [
            {
                test: /\.wav$/,
                use: require.resolve('file-loader')
            }
        ]
    }
}
```

加载文件，可以设置阈值，小于阈值时把文件转为base64编码，这样无需再从服务器拉取图片

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
                            name: 'img/[name].[ext]'
                        }
                    }
                ]
            }
        ]
    }
}
```

- image-loader

加载并压缩图片

- babel-loader

es6+转为es5

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
                            ...babelOptions
                        }
                    }
                ]
            }
        ]
    }
}
```

- ts-loader

ts转为js

- css-loader

处理@import或url引入的外部资源

- style-loader

创建style标签引入样式

```js
const config = {
    // ...
    module: {
        rules: [
            {
                test: /\.css$/,
                use: [
                    'style-loader',
                    'css-loader'
                ]
            }
        ]
    }
}
```

- postcss-loader

扩展css语法，可以使用autoprefixer（自动添加不同浏览器下的样式前缀）等插件

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
                            sourceMap: true
                        }
                    }
                ]
            }
        ]
    }
}

// postcss.config.js
const autoprefixer = require('autoprefixer');

module.exports = {
    plugins: [
        autoprefixer({
            // 目前只有发包的构建使用到postcss,所以直接将env设置为production
            env: 'production'
        })
    ]
};
```

- cache-loader

对一些性能开销比较大的loader前添加cache-loader，将其结果缓存到磁盘

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
                        configFile: path.join(__dirname, '../eslint/eslintrc.dev.js')
                    }
                }
            }
        ]
    }
};
```