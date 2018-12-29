> 在 ES6 里面，新增了两个常用的申明变量的关键字 let 和 const，旨在替换 var 关键字，毕竟 var 申明的变量会被提升到全局作用域，污染了全局环境，并会出现一些奇怪的问题。

1. let 命令基本用法

****

ES6 新增了 let 命令，用来申明变量。它的用法类似于 var, 但是所申明的变量，只在 let 命令所在的代码块内有效。

```
{
  let a = 10;
  var b = 1;
}

a // ReferenceError: a is not defined.
b // 1
```

上面代码在代码快中，分别用 let 和 var 申明了两个变量。然后在代码块之外调用这两个变量，结果 let 声明的变量报错， var 声明的变量返回了正确的值，这表明 let 声明的变量只在它所在的代码块有效。

以下代码展示在 `for` 循环中 let 和 var 声明变量的区别。

```
// var 栗子

var a = [];
for (var i = 0; i < 10; i++) {
  a[i] = function () {
    console.log(i);
  };
}
a[6](); // 10

// let 栗子
var a = [];
for (let i = 0; i < 10; i++) {
  a[i] = function () {
    console.log(i);
  };
}
a[6](); // 6

```

