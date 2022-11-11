#### 为什么需要 WebAssembly

v8 执行一段 js 代码的过程是这样的，解析器将 js 源码转为 AST，解释器将 AST 转为字节码然后执行，遇到热点代码编译器直接将其转为机器码执行，下次再执行相同代码时可以直接执行机器码。

但是 JIT 优化只针对静态类型的变量，而 javascript 是动态类型的语言，如果在函数执行过程中类型发生变化就会导致 JIT 失效，需要重新进行解析-解释-执行的过程，比较耗时。退一步讲就算控制了类型不随意变化从而充分利用 JIT，但是其他非热点代码仍然需要解析的过程

为了极致的性能，诞生了汇编语言 WebAssembly，够进行最大限度的 JIT 优化，使得 WebAssembly 的速度能够无限逼近 C/C++ 等原生代码，同时兼具体积小、性能高、可移植性强等特点。但这并不意味着 WebAssembly 会取代 JavaScript，它桥接了多种编程语言的生态，对 JavaScript 做了功能和性能方面的补足

#### WebAssembly 使用

WebAssembly 对应的 WASM 是一种低级的类汇编语言，通过 wabt 等工具可以将其转为二进制代码，示例如下

```wasm
(module
  (func $i (import "imports" "imported_func") (param i32))
  (func (export "exported_func")
    i32.const 42
    call $i
  )
)
```

Emscripten 编译器可以将 C/C++代码编译成 JS 胶水代码、 WASM 和加载 WASM 的 HTML，其核心工具为 Emscripten Compiler Frontend（emcc）。通过 JS 胶水代码将 WASM 跑在浏览器或 node 中，JS 胶水代码主要实现了映射到 C/C++ 相关操作的 Web API

浏览器中提供了 [WebAssembly 命名空间](https://developer.mozilla.org/zh-CN/docs/WebAssembly/JavaScript_interface)，提供了方法可以执行 wasm 代码，主要有以下几个概念

- 模块：表示一个已经被浏览器编译为可执行机器码的 WebAssembly 二进制代码
- 内存：ArrayBuffer，大小可变，WebAssembly 的低级内存存取指令可以对它进行读写操作
- 表格：带类型数组，存储了不能作为原始字节存储在内存里的对象的引用
- 实例：一个模块及其在运行时使用的所有状态，包括内存、表格和一系列导入值

#### 如何编译 FFmpeg 到 WebAssembly

像 FFmpeg 这样的大型的项目依赖于 C 标准库、操作系统、文件系统或其他依赖，这些项目在编译前依赖 autoconfig/automake 等库来生成系统特定的代码，Emscripten 提供了 emconfigure 和 emmake 来封装这些命令

```bash
./configure # 处理前置依赖
make # 使用 gcc 等进行编译构建，生成对象文件
```

使用 Emscripten 编译大部分复杂的 C/C++ 库时会经历以下几个步骤

1. 安装特定依赖

```bash
git clone https://github.com/emscripten-core/emsdk.git
cd emsdk
# 安装 1.39.18 版本的 Emscripten
./emsdk install 1.39.18
./emsdk activate 1.39.18
source ./emsdk_env.sh
# n4.3.1 的 ffmpeg
git clone --depth 1 --branch n4.3.1 https://github.com/FFmpeg/FFmpeg
```

2. 使用 emconfigure 处理 configure 文件，主要是替换 gcc 编译器为 emcc

```bash
export CFLAGS="-s USE_PTHREADS -O3"

export LDFLAGS="$CFLAGS -s INITIAL_MEMORY=33554432"

emconfigure ./configure \

  --target-os=none \ # 设置为 none 来去除特定操作系统的一些依赖

  --arch=x86_32 \ # 选中架构为 x86_32

  --enable-cross-compile \ # 处理跨平台操作

  --disable-x86asm \  # 关闭 x86asm

  --disable-inline-asm \  # 关闭内联的 asm

  --disable-stripping \ # 关闭处理 strip 的功能，避免误删一些内容

  --disable-programs \ # 加速编译

  --disable-doc \  # 添加一些 flag 输出

  --extra-cflags="$CFLAGS" \

  --extra-cxxflags="$CFLAGS" \

  --extra-ldflags="$LDFLAGS" \

  --nm="llvm-nm" \  # 使用 llvm 的编译器

  --ar=emar \

  --ranlib=emranlib \

  --cc=emcc \ # 将 gcc 替换为 emcc

  --cxx=em++ \ # 将 g++ 替换为 em++

  --objcc=emcc \

  --dep-cc=emcc
```

3.  构建依赖

使用 emmake make

```bash
emmake make -j4
```

也可以使用 emcc 进行自定义编译，最终生成 ffmpeg-core.js、ffmpeg-core.wasm、ffmpeg-core.worker.js 三个文件

```bash
mkdir -p wasm/dist

emcc \

 -I. -I./fftools \

  -Llibavcodec -Llibavdevice -Llibavfilter -Llibavformat -Llibavresample -Llibavutil -Llibpostproc -Llibswscale -Llibswresample \

  -Qunused-arguments \

  -o wasm/dist/ffmpeg-core.js fftools/ffmpeg_opt.c fftools/ffmpeg_filter.c fftools/ffmpeg_hw.c fftools/cmdutils.c fftools/ffmpeg.c \

  -lavdevice -lavfilter -lavformat -lavcodec -lswresample -lswscale -lavutil -lm \

  -O3 \

  -s USE_SDL=2 \    # 使用 SDL2

  -s USE_PTHREADS=1 \

  -s PROXY_TO_PTHREAD=1 \ # 将 main 函数与浏览器/UI主线程分离

  -s INVOKE_RUN=0 \ # 执行 C 函数时不首先执行 main 函数

  -s EXPORTED_FUNCTIONS="[_main, _proxy_main]" \ # 导出 ffmpeg 对应的 C 文件里的 main 函数

  -s EXTRA_EXPORTED_RUNTIME_METHODS="[FS, cwrap, setValue, writeAsciiToMemory]" \ # 导出一些 runtime 的辅助函数

  -s INITIAL_MEMORY=33554432
```

4. 使用编译完成的 ffmpeg wasm 模块

```js
const Module = require('./dist/ffmpeg-core.js');

// 加载 WebAssembly 模块完成之后执行的逻辑
Module.onRuntimeInitialized = () => {
  // 导出 C 文件中的 proxy_main 使用
  const ffmpeg = Module.cwrap('proxy_main', 'number', ['number', 'number']);
  const args = ['ffmpeg', '-hide_banner'];

  const argsPtr = Module._malloc(args.length * Uint32Array.BYTES_PER_ELEMENT);

  args.forEach((s, idx) => {
    // // 额外分配一个字节的空间来存放 0 表示字符串的结束
    const buf = Module._malloc(s.length + 1);
    // 将 JavaScript 的字符串转换成 C 中的字符数组
    Module.writeAsciiToMemory(s, buf);
    // 创建 C 中的 32 位整数的指针数组
    Module.setValue(argsPtr + Uint32Array.BYTES_PER_ELEMENT * idx, buf, 'i32');
  });

  ffmpeg(args.length, argsPtr);
};
```

Emscripten 内建了一个虚拟的文件系统 FS 来支持 C 中标准的文件读取和写入 Uint8Array 数据

```js
const fs = require('fs');
const Module = require('./dist/ffmpeg-core');

Module.onRuntimeInitialized = () => {
  const data = Uint8Array.from(fs.readFileSync('./flame.avi'));
  Module.FS.writeFile('flame.avi', data);
};
```

如果想在浏览器中用 ffmpeg 转码视频并播放，需要使用 libx264 编码器来将 mp4 文件编码成浏览器可播放的编码格式

#### 调试

C/C++ Devtools Support 浏览器扩展支持 Chrome 开发者工具调试 C/C++ 代码，该扩展会解析 DWARF 调试文件，为 Chrome Devtools 在调试时提供 source map 相关的信息

参考

1. [为什么说 WebAssembly 是 Web 的未来？](https://juejin.cn/post/7056612950412361741)
2. [WebAssembly 概念](https://developer.mozilla.org/zh-CN/docs/WebAssembly/Concepts)
