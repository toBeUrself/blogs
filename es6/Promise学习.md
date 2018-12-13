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
executor是带有 resolve 和 reject 两个参数的函数 。Promise构造函数执行时立即调用executor 函数， resolve 和 reject 两个函数作为参数传递给executor（executor 函数在Promise构造函数返回新建对象前被调用）。resolve 和 reject 函数被调用时，分别将promise的状态改为fulfilled（完成）或rejected（失败）。executor 内部通常会执行一些异步操作，一旦完成，可以调用resolve函数来将promise状态改成fulfilled，或者在发生错误时将它的状态改为rejected。
如果在executor函数中抛出一个错误，那么该promise 状态为rejected。executor函数的返回值被忽略。
