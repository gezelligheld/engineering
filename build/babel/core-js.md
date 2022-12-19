core-js会对新的方法如es6的flatMap、promise等进行polyfill，直接全部注入到全局环境里面

#### 分类

core-js目前最新版是v3，分为三种版本

- core-js

只要在项目的入口文件中，引入core-js一次，就能把所有的polyfill注入到运行环境中

```js
import 'core-js'; // <- at the top of your entry point
```

也可以按需引入

```js
import 'core-js/features/array/from'; // <- at the top of your entry point
import 'core-js/features/array/flat'; // <- at the top of your entry point
import 'core-js/features/set';        // <- at the top of your entry point
import 'core-js/features/promise';    // <- at the top of your entry point
```

- core-js-pure

不会把polyfill注入全局环境，在使用的时候，需要单独引入polyfill的module

```js
import arrayfrom from 'core-js-pure/features/array/from';
import flat from 'core-js-pure/features/array/flat';
import Set from 'core-js-pure/features/set';
import Promise from 'core-js-pure/features/promise';
```

- core-js-bundle

一个编译打包好的版本，包含全部的polyfill特性，适合在浏览器里面通过script直接加载

#### core-js的modules组织方式

```js
// polyfill all `core-js` features - ES and web standards:
import "core-js"; // equivalent to `import "core-js/es; import "core-js/web; import "core-js/proposals"`;

// polyfill all stable `core-js` features - ES and web standards:
import "core-js/stable"; // equivalent to `import "core-js/es; import "core-js/web";

// polyfill all stable `core-js` features - only ES:
import "core-js/es";

// polyfill all stable `core-js` features - only web standards:
import "core-js/web";

// polyfill all `core-js` proposal features - ES and web standards:
import "core-js/proposals";

// polyfill all `core-js` proposal features - ES and web standards:
import "core-js/stage"; // equivalent to ` import "core-js/proposals"

// polyfill all `core-js` stage0+ proposal features - ES and web standards:
import "core-js/stage/0";

// polyfill all `core-js` stage1+ proposal features - ES and web standards:
import "core-js/stage/1";

// polyfill all `core-js` stage2+ proposal features - ES and web standards:
import "core-js/stage/2";

// polyfill all `core-js` stage3+ proposal features - ES and web standards:
import "core-js/stage/3";

// polyfill all `core-js` stage4+ proposal features - ES and web standards:
import "core-js/stage/4";

// polyfill all `core-js` features - ES and web standards:
import "core-js/features"; // equivalent to ` import "core-js";
```

#### core-js源码

所有core-js的polyfill，底层的机制都是由internals和modules内部的module来完成的，并没有提供命名空间，不可以直接引入，因为这些内部实现在迭代中可能随时变化

core-js 通过向原型属性prototype或全局注入方法来实现polyfill

(源码地址)[https://github.com/zloirock/core-js/tree/master/packages/core-js]