1. demo1

```js
let a = 1;

// 编译后
var a = 1;
```

2. demo2

```js
if (false) {
    let value = 1;
}
console.log(value); // Uncaught ReferenceError: value is not defined

// 编译后
if (false) {
    let _value = 1;
}
console.log(value);
```

3. demo3

循环中的let使用形参存储每次迭代的值

```js
var funcs = [];
for (let i = 0; i < 10; i++) {
    funcs[i] = function () {
        console.log(i);
    };
}
funcs[0](); // 0

// 编译后
var funcs = [];

var _loop = function _loop(i) {
    funcs[i] = function () {
        console.log(i);
    };
};

for (var i = 0; i < 10; i++) {
    _loop(i);
}
funcs[0](); // 0
```