# Promise 的前世今生

[Promise 必知必会十道题测试](https://zhuanlan.zhihu.com/p/30828196)

>Promise 的含义
Promise 是**异步编程**的一种解决方案，比传统的解决方案——回调函数和事件——更合理强大。它由社区最早提出和实现，ES6 将它写进了语言标准，统一了用法，原生提供了`Promise` 对象。

`Promise` 这个对象是一个可以获取异步操作消息的容器，它提供统一的API，各种异步操作都可以用同样的方法进行处理。它有以下两个特点：
1. 对象的状态不受外界的影响。`Promise` 对象代表一个异步操作，有三种状态:`pending` [进行中]、`fulfilled` [已成功] 和 `Rejected` [已失败]。只有异步操作的结果可以决定当前是哪一种状态，任何其它操作都无法改变这个状态。这也是 `Promise` 这个名字的由来**承诺**；
2. 状态一旦改变，就会固定，不会再变，并且任何时候都可以得到这个结果[并且可以多次获取同一结果]。`Promise` 对象的状态改变只有两种情况：从 `pending` 变为 `fulfilled` 和从 `pending` 变为 `rejected`。只要其中一种情况发生，状态就凝固了，不会再变，会一直保持这个结果，这时就称为 resolved (已定型)。如果状态改变后，Promise 对象有添加 then，就会立即得到结果。这与事件(Event)完全不同，事件的特点是，如果你错过了它，再去监听，是不能得到结果的。

有了 `Promise` 对象，就可以将异步操作以同步操作的流程表达出来，避免了层层嵌套的回调函数。此外，`Promise` 对象提供统一的接口，使得控制异步操作更加容易。

`Promise` 也有缺点：
- 无法取消 Promise。一旦新建它就会立即执行，无法中途取消。
- 如果不设置回调函数，Promise 内部抛出的错误，不会反应到外部。
- 当处于 `pending` 状态时，无法得知目前进展到哪一个阶段(刚刚开始还是即将结束)

**如果某些事件不断的发生，一般来说， 使用 [Stream](https://nodejs.org/api/stream.html) 模式是比部署 `Promise` 更好的选择。**

>Promise 的用法

`new Promise( function(resolve, reject) {...} /* executor */  );`

executor
executor是带有 resolve 和 reject 两个参数的函数 。Promise构造函数执行时立即调用executor 函数， resolve 和 reject 两个函数作为参数传递给executor（executor 函数在Promise构造函数返回新建对象前被调用）。resolve 和 reject 函数被调用时，分别将promise的状态改为fulfilled（完成）或rejected（失败）。executor 内部通常会执行一些异步操作，一旦完成，可以调用resolve函数来将promise状态改成fulfilled，或者在发生错误时将它的状态改为rejected。`Promise` 构造函数接受一个函数作为参数，该函数的两个参数分别是**resolve**和**reject**。它们是两个函数，由 JavaScript 引擎提供，不用自己部署。如果在executor函数中抛出一个错误，那么该promise 状态为rejected。executor函数的返回值被忽略。Promise实例生成以后，可以用then方法分别指定resolved状态和rejected状态的回调函数。

then方法可以接受两个函数作为参数，第一个函数作为resolve状态的回调，第二个函数作为reject状态的回调。***但是第二个参数一般不写在 then 方法里面，而是单独写一个 catch 方法，作为 reject 状态的回调。捕捉异常，而且 Promise 的异常具有冒泡属性，catch 总是能捕捉到前面的异常，但是无法捕获后面的异常。***

当resolve的返回参数是另外一个Promise的时候
```
const p1 = new Promise(function (resolve, reject) {
  // ...
});

const p2 = new Promise(function (resolve, reject) {
  // ...
  resolve(p1);
})
```
如下需要注意：

***这时p1的状态就会传递给p2，也就是说，p1的状态决定了p2的状态。如果p1的状态是pending，那么p2的回调函数就会等待p1的状态改变；如果p1的状态已经是resolved或者rejected，那么p2的回调函数将会立刻执行。***

>then 方法： Promise.protype.then
Promise 实例具有then方法，也就是说，then方法是定义在原型对象Promise.prototype上的。它的作用是为 Promise 实例添加状态改变时的回调函数。前面说过，then方法的第一个参数是resolved状态的回调函数，第二个参数（可选）是rejected状态的回调函数。catch就是第二个参数的变体。

then方法返回的是一个新的Promise实例（注意，不是原来那个Promise实例）。因此可以***采用链式写法***，即then方法后面再调用另一个then方法。

***因为Promise的状态一旦改变就不会再变，所以如果promise中同时存在 resolve，reject 和 throw Error。promise 也只会是第一个改变状态后的情况，再之后的状态改变对promise无效。***

在promise中抛出异常，会将 promise 的状态改变成 rejected. 而且此异常只会在promise 的then的第二个参数 和 catch 中被捕获。不会被抛出到外面。

>但是下面这种情况需要注意
```
const promise = new Promise(function (resolve, reject) {
  resolve('ok');
  setTimeout(function () { throw new Error('test') }, 0)
});
promise.then(function (value) { console.log(value) });
```
***上面代码中，Promise 指定在下一轮“事件循环”再抛出错误。到了那个时候，Promise 的运行已经结束了，所以这个错误是在 Promise 函数体外抛出的，会冒泡到最外层，成了未捕获的错误。***

此外，promise还有一些辅助函数，这里不再赘述！
