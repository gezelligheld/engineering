package.json 文件定义了运行项目所需要的各种依赖和项目的配置信息，主要属性如下

#### 必填属性

##### name

定义了模块的名称，命名时需要注意以下几点

- 模块名会成为模块 url、命令行中的一个参数或者一个文件夹名称，任何非 url 安全的字符在模块名中都不能使用
- 若模块名称中存在一些符号，将符号去除后不得与现有的模块名重复，例如：由于 react-router-dom 已经存在，react.router.dom、reactrouterdom 都不可以再创建
- 不能与其他模块名重复

```bash
# 查看模块基本信息
npm view <packageName>
```

##### version

使用 x.y.z 形式，对应 主版本.次版本.修订版本

- 主版本：当你做了不兼容的 API 修改
- 次版本：当你做了向下兼容的功能性新增
- 修订版本：当你做了向下兼容的问题修正

x.y.z 格式是模块正式版本，重要模块为了保证稳定，会在放出正式版本之前提供先行版本，使用-连接

- alpha: 内部版本
- beta: 公测版本
- rc：正式版本的候选版本

模块依赖的版本号有些几种不同写法，来保证安装的时候使用对应的版本

- x.y.z： 使用精确版本号
- \*： 任意版本，第一次安装会使用模块最新版本
- ^x.y.z x： 位锁死，y、z 位使用最新版本
- 3.x： 和 ^3.0.0 含义相同
- ~x.y.z：x、y 锁定，z 位使用最新版本

```bash
npm view <packageName> version # 查看某个模块的最新版本
npm view <packageName> versions # 查看某个模块的所有历史版本
```

#### 描述信息

使用 npm 检索模块时，会对模块中的 description 字段和 keywords 字段进行匹配，可以增加曝光率

- description：添加模块的描述信息
- keywords：给模块添加关键字

#### 依赖属性

- dependencies：生产环境模块依赖
- devDependencies：开发环境模块依赖
- peerDependencies：与宿主模块共享依赖，或自己不安装而希望宿主环境安装的时候使用 peerDependencies 声明

```bash
# 写入 dependencies 属性
npm install <package...>
npm install <package...> --save

# 写入 devDependencies 属性
npm install <package...> --save-dev
npm install <package...> -D
```

#### 简化终端命令

scripts 字段定义的对象属性值可以通过 npm run 运行脚本

#### 定义项目入口

main 字段用来指定加载的入口文件，当不指定 main 字段时，默认值是模块根目录下面的 index.js 文件。假如你的项目是一个 npm 包，当用户安装你的包后，require('my-module') 返回的是 main 字段中所列出文件的 module.exports 属性

#### 发布文件配置

files 字段用于描述我们使用 npm publish 命令后推送到 npm 服务器的文件列表

此外可以通过.npmignore 来排除一些文件， 防止大量的垃圾文件推送到 npm 上

#### 定义私有模块

一般公司的非开源项目，都会设置 private 属性的值为 true，防止私有模块被无意间发布出去

#### 指定模块适用系统和 cpu

os 属性可以指定模块适用系统的系统，或者指定不能安装的系统黑名单

```bash
"os" : [ "darwin", "linux" ] # 适用系统
"os" : [ "!win32" ] # 黑名单
```

cpu 属性用来限制 cpu，用法相同

```bash
"cpu" : [ "x64", "ia32" ] # 适用 cpu
"cpu" : [ "!arm", "!mips" ] # 黑名单
```

#### 指定项目 node 版本

有时候，新拉一个项目的时候，由于和其他开发使用的 node 版本不同，导致会出现很多奇奇怪怪的问题，这就需要制定 node 版本。它只起到说明作用，不符合指定的版本也能安装

```json
"engines": {
   "node": ">= 8.16.0"
},
```

#### 自定义命令

bin 字段用来指定各个内部命令对应的可执行文件的位置，即相当于做了一个命令名和本地文件名的映射

当用户安装带有 bin 字段的包时

- 如果是全局安装，npm 将会使用符号链接把这些文件链接到/usr/local/node_modules/.bin/
- 如果是本地安装，会链接到./node_modules/.bin/

写法如下

```bash
"bin": {
  "my-app-cli": "./bin/cli.js"
}

# 或，key默认是模块名
"bin": "./bin/cli.js"
```

其中指定的./bin/cli.js 需要在第一行写入以下命令，才能使用命令 my-app-cli

```js
#! /usr/bin/env node
```

#### 自定义字段

此外还可以自定义字段，以满足不同环境下的构建需求

参考

1. [重新认识 package.json](https://juejin.cn/post/6844904159226003463)
