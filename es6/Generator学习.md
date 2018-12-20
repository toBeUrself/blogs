# Generator 的语法以及异步应用

>ES6 中为了更方便地写异步代码，在标准中引入了 `Async` 和 `Await`。而这么高大上的东西，其实就是 `Generator` 函数的语法糖。那么 Generator 是什么？

1. 基本概念

`Generator` 函数和 `Promise` 一样，也是ES6提供的一种异步编程的解决方案，语法行为与传统的函数完全不同。

**在语法上，可以把它理解成一个状态机，封装了多个内部状态。与其他函数不同的是，执行 Generator 函数不会返回结果，而是会返回一个遍历器，就是说，Generator除了是一个状态机，还是一个遍历器对象生成函数。返回的这个遍历器对象，可以依次遍历 Generator 函数内部的每一个状态。**

**在形式上，比普通函数多了两个特征。一是，`function` 关键字与函数名之间有一个星号；二是，函数体内部使用 `yield` 表达式，定义不同的内部状态。**

***栗子***
```javascript
function* FirstGenerator() {
  yield 'hello';
  yield 'world';
  return 'ending';
}

var g = FirstGenerator();
g.next()
// { value: 'hello', done: false }
g.next()
// { value: 'world', done: false }
g.next()
// { value: 'ending', done: true }
g.next()
// { value: undefined, done: true }
```
