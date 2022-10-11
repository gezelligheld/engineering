#### 概念

用于扩展 webpack 的功能。webpack 会在构建过程中广播出很多事件，plugin 可以在恰当的时机做更多的事情，如代码压缩、定义环境变量等

#### 常用 plugin

- webpack.IgnorePlugin

忽略第三方包指定目录，让这些指定目录不要被打包进去

```js
module.exports = {
  // ...
  plugins: [
    new webpack.IgnorePlugin({
      resourceRegExp: /^\.\/locale$/,
      contextRegExp: /moment$/,
    }),
  ],
};
```

- terser-webpack-plugin

压缩 js

```js
module.exports = {
  optimization: {
    minimize: true,
    minimizer: [new TerserPlugin()],
  },
};
```

- webpack.DefinePlugin

编译时创建全局变量

```js
module.exports = {
  // ...
  plugins: [
    new webpack.DefinePlugin({
      BROWSER_ENV: JSON.stringify(process.env.BROWSER_ENV),
    }),
  ],
};
```

- webpack.HotModuleReplacementPlugin

开启热更新

- html-webpack-plugin

根据模版自动生成 html、自动引入 css 和 js

```js
module.exports = {
  // ...
  plugins: [
    new HtmlWebpackPlugin({
      template: path.resolve(__dirname, '../template/index.pug'),
      filename: '../pages/index.html',
      favicon: path.resolve(__dirname, '../images/favicon-32.ico'),
      templateParameters: {
        mock: true,
      },
    }),
  ],
};
```

- mini-css-extract-plugin

用于抽离 css 样式为 link 标签引入

```js
module.exports = {
  // ...
  plugins: [
    new MiniCssExtractPlugin({
      filename: 'css/[name]-[contenthash:8].css',
    }),
  ],
};
```

- webpack.DllPlugin

先把常用但又构建时间长的代码提前打包好（例如 react、react-dom），取个名字叫 dll，后面再打包的时候就跳过原来的未打包代码，直接用 dll，提升了打包速度

vue 和 react 脚手架都摒弃了 dll 的配置，webpack4 的打包速度足够快，dll 的加速不明显了

```js
module.exports = {
  // ...
  plugins: [
    new webpack.DllPlugin({
      path: path.join(context, toolsPath, '[name]_manifest.json'),
      name: '[name]_library',
      context: path.join(context, toolsPath),
    }),
  ],
};
```

- webpack-bundle-analyzer

可视化 Webpack 输出文件的体积

#### loader 和 plugin 的区别

- loader：对其他类型的资源进行转译的预处理工作，编译为 js 代码

- plugin：可以扩展 Webpack 的功能，在 Webpack 运行的生命周期中会广播出许多事件，Plugin 可以监听这些事件，在合适的时机通过 Webpack 提供的 API 改变输出结果

#### 自定义 plugin

自定义 plugin 是一个含有接收 complier 实例作为参数的 apply 方法的类，可以订阅构建过程中抛出来的事件，示例如下

```js
// FileListPlugin.js
class FileListPlugin {
  apply(compiler) {
    compiler.plugin('emit', function (compilation, callback) {
      var filelist = 'In this build:\n\n';
      for (var filename in compilation.assets) {
        filelist += '- ' + filename + '\n';
      }
      compilation.assets['filelist.md'] = {
        source: function () {
          return filelist;
        },
        size: function () {
          return filelist.length;
        },
      };
      // 有些事件是异步的，这些异步的事件会附带两个参数，第二个参数为回调函数，在插件处理完任务时需要调用回调函数通知 webpack，才会进入下一处理流程
      callback();
    });
  }
}
module.exports = FileListPlugin;
// webpack.config.js
module.exports = {
  plugins: [new FileListPlugin({ param: 'xxx' })],
};
```

参考文档

1. [你真的需要 DllPlugin 吗](https://www.bilibili.com/read/cv6118617/)
