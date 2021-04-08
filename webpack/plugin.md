#### 概念

插件可以用于执行范围更广的任务，可以打包优化和压缩、定义环境变量等

#### 常用plugin

- webpack.IgnorePlugin

忽略第三方包指定目录，让这些指定目录不要被打包进去

```js
module.exports = {
    // ...
    plugins: [
        new webpack.IgnorePlugin({
            resourceRegExp: /^\.\/locale$/,
            contextRegExp: /moment$/
        })
    ]
}
```

- terser-webpack-plugin

压缩js

```js
module.exports = {
  optimization: {
    minimize: true,
    minimizer: [new TerserPlugin()],
  }
};
```

- webpack.DefinePlugin

编译时创建全局变量

```js
module.exports = {
    // ...
    plugins: [
        new webpack.DefinePlugin({
            BROWSER_ENV: JSON.stringify(process.env.BROWSER_ENV)
        })
    ]
}
```

- webpack.HotModuleReplacementPlugin

开启热更新

- html-webpack-plugin

根据模版自动生产html、自动引入css和js

```js
module.exports = {
    // ...
    plugins: [
        new HtmlWebpackPlugin({
            template: path.resolve(__dirname, '../template/index.pug'),
            filename: '../pages/index.html',
            favicon: path.resolve(__dirname, '../images/favicon-32.ico'),
            templateParameters: {
                mock: true
            }
        })
    ]
}
```

- mini-css-extract-plugin

用于抽离css样式为link标签引入

```js
module.exports = {
    // ...
    plugins: [
        new MiniCssExtractPlugin({
            filename: 'css/[name]-[contenthash:8].css'
        })
    ]
}
```

- webpack.DllPlugin

先把常用但又构建时间长的代码提前打包好（例如 react、react-dom），取个名字叫 dll，后面再打包的时候就跳过原来的未打包代码，直接用 dll，提升了打包速度

vue和react脚手架都摒弃了dll的配置，webpack4的打包速度足够快，dll的加速不明显了

```js
module.exports = {
    // ...
    plugins: [
        new webpack.DllPlugin({
            path: path.join(context, toolsPath, '[name]_manifest.json'),
            name: '[name]_library',
            context: path.join(context, toolsPath)
        })
    ]
}
```

参考文档
1. [你真的需要DllPlugin吗](https://www.bilibili.com/read/cv6118617/)