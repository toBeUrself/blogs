# Generator 函数的异步应用

> 异步编程对 Javascript 语言太重要。JavaScript 语言的执行环境是‘单线程’的，如果没有异步编程，根本没法用，要卡死的。

1. 传统方法

ES6 诞生以前，异步编程的方法大概有一下四种。

- 回调函数
- 事件监听
- 发布、订阅
- Promise 对象

Generator 函数将 JavaScript 异步编程带入了一个全新的阶段。

****

2. 基本概念

所谓‘异步’，就是一个任务不是连续完成的，可以理解成该任务被人分为两个阶段，先是执行第一段，然后转而执行其他任务，等做好了准备，再回头执行第二段。

栗子：有一个任务是读取文件进行处理，第一段是向操作系统发出请求，要求读取文件。然后程序执行其他任务，等到操作系统返回文件，再接着执行任务的第二段（处理文件）。这种不连续的执行，就是异步。

相应的，连续执行的任务就是同步任务。由于是连续执行，不能插入其他任务，所以操作系统从硬盘读取文件这段时间，程序只能干等着。

****

3. 回调函数

JavaScript 语言对异步编程的实现，就是回调函数。所谓回调函数，就是把任务的第二段单独写在一个函数里面，等到重新执行这个任务的时候，就直接调用这个函数。回调函数的英文名字 callback，直译过来就是‘重新调用’。

读取文件经行处理，如下：

```
fs.readFile('./etc/passwd', 'utf-8', function(err, data) {
    if(err) throw err;
    console.log(data);
```

上面第三个参数就是回调函数。
+ 一个有趣的问题是，为什么 Node 约定，回调函数的第一个参数，必须是错误对象err（如果没有错误，该参数就是null）？

+ 原因是执行分成两段，第一段执行完以后，任务所在的上下文环境就已经结束了。在这以后抛出的错误，原来的上下文环境已经无法捕捉，只能当作参数，传入第二段。

****

4. Promise 函数

Promise 函数可以避免回调函数陷入回调地狱。这不是新的语法功能，而是一种新的写法，允许将回调函数的嵌套，改成链式调用。比如读取多个文件，用promise写法如下：

```
var readFile = require('fs-readfile-promise');

readFile(fileA)
.then(function (data) {
  console.log(data.toString());
})
.then(function () {
  return readFile(fileB);
})
.then(function (data) {
  console.log(data.toString());
})
.catch(function (err) {
  console.log(err);
});
```

Promise 的最大问题是代码冗余，原来的任务被 Promise 包装了一下，不管什么操作，一眼看去都是一堆then，原来的语义变得很不清楚

****

3. Generator 函数

+ 协程

传统的函数语言，早有异步编程的解决方案（其实是多任务的解决方案）。其中一种叫做‘协程’，意思是多个线程相互协作，完成异步任务。

协程有点像函数，又有点像线程。它的运行流程大致如下。

- 第一步，协程 A 开始执行
- 第二步，协程 A 执行到一半，进入暂停，执行权转移到协程 B
- 第三步，（一段时间后），协程 B 交还执行权
- 第四步，协程 A 恢复执行

上面流程的协程 A，就是异步任务，因为它分成两段（或多段）执行。

**栗子**

```
function* asyncJob(){
  // ...
  let f = yield readFile(fileA);
  // ...
```

上面代码的函数 `asyncJob` 就是一个协程，它的奥妙就在其中的yield命令。他表示执行到此处，执行权将交给其他协程。也就是说，yield 命令是异步两个分界线。

协程遇到yield命令就暂停，等到执行权返回，再从暂停的地方继续往后执行。它的最大优点就是代码的写法非常像同步操作，如果去除yield命令，简直就是一模一样。

+ Generator 函数可以暂停执行和恢复执行，这是他能封装异步任务的根本原因。除此之外，他还有两个特性，使他可以作为异步编程的完整解决方案：函数体内外的数据交换和错误处理机制。

**** 

4. 异步任务的封装

下面看看如何使用一个 Generator 函数执行一个真实的异步任务。

```
var fetch = require('node-fetch');

function* gen(){
  var url = 'https://api.github.com/users/github';
  var result = yield fetch(url);
  console.log(result.bio);
}

var g = gen();
var result = g.next();

result.value.then(function(data){
   g.next(data.json());
});
```

上面代码中，首先执行 Generator 函数，获取遍历器对象，然后使用next方法（第二行），执行异步任务的第一阶段。由于Fetch模块返回的是一个 Promise 对象，因此要用then方法调用下一个next方法。

可以看到，虽然 Generator 函数将异步操作表示得很简洁，但是流程管理却不方便（即何时执行第一阶段、何时执行第二阶段）。

****

5. Thunk 函数

>Thunk 函数是自动执行 Generator 函数的一种方法。

参数的求值策略

Thunk 函数早在上个世纪60年代就诞生了。

那时，编程语言刚刚起步，计算机科学家还在研究编译器怎么写比较好，一个争论的焦点是‘求值策略’，即函数的参数到底应该何时求值。

```
const x = 1;

function f(m) {
  return m * 2;
}

f(x + 5)
```

上面代码先定义函数f，然后向他传入表达式`x+5`。请问，这个表达式应该何时求值？

一种意见是‘传值调用’（call by value），即在进入函数体之前，就计算x+5的值，再将这个结果传入函数f。C语言就采用这种策略。

另一种意见是‘传名调用’，即直接将表达式x+5传入函数体，只在用到它的时候求值。Hashkell语言采用这种策略。

```
function f(a, b){
  return b;
}

f(3 * x * x - 2 * x - 1, x);
```

+ 上面代码中，函数`f`的第一个参数是一个复杂的表达式，但是函数体内部没有用到。对这个参数求值是不必要的。因此，有一些计算机科学家倾向于“传名调用”，即只在执行时求值。

**Thunk 函数的含义**

编译器的“传名调用”实现，往往是将参数放到一个临时函数中，在将这个临时函数传入函数体。这个临时函数就是 Thunk 函数。

```
function f(m) {
  return m * 2;
}

f(x + 5);

// 等同于

var thunk = function () {
  return x + 5;
};

function f(thunk) {
  return thunk() * 2;
}
```

+ 上面的代码中，函数 f 的参数 `x+5` 被一个函数替换了。凡是用到原参数的地方，对 Thunk 函数求值即可。这就是 Thunk 函数的定义，他是“传名调用”的一种实现策略，用来替换某个表达式。

****

6. JavaScript 语言的 Thunk 函数

JavaScript 语言是传值调用，它的 Thunk 函数含义有所不同。在 JavaScript 语言中， Thunk 函数替换的不是表达式，而是多参数函数，将其替换成一个只接受回调函数作为参数的单参数函数。

```
// 正常版本的readFile（多参数版本）
fs.readFile(fileName, callback);

// Thunk版本的readFile（单参数版本）
var Thunk = function (fileName) {
  return function (callback) {
    return fs.readFile(fileName, callback);
  };
};

var readFileThunk = Thunk(fileName);
readFileThunk(callback);
```

上面代码中，fs模块的readFile方法是一个多参数函数，两个参数分别为文件名和回调函数。经过转换器处理，它变成了一个单参数函数，只接受回调函数作为参数。这个单参数版本，就叫做 Thunk 函数。

任何函数，只要参数有回调函数，就能写成 Thunk 函数的形式。下面是一个简单的 Thunk 函数转换器

```
// ES5版本
var Thunk = function(fn){
  return function (){
    var args = Array.prototype.slice.call(arguments);
    return function (callback){
      args.push(callback);
      return fn.apply(this, args);
    }
  };
};

// ES6版本
const Thunk = function(fn) {
  return function (...args) {
    return function (callback) {
      return fn.call(this, ...args, callback);
    }
  };
};
```

使用上面的转换器，生成fs.readFile的 Thunk 函数

```
var readFileThunk = Thunk(fs.readFile);
readFileThunk(fileA)(callback);

// 另外一个例子
function f(a, cb) {
  cb(a);
}
const ft = Thunk(f);

ft(1)(console.log) // 1
```
****

7 Generator 函数的流程管理

Thunk 函数在之前其实没有什么用处，但是 ES6 有了 Generator 函数之后，Thunk 函数可以用于 Generator 函数的自动流程管理。

**栗子：Generator 函数可以自动执行**

```
function* gen() {
  // ...
}

var g = gen();
var res = g.next();

while(!res.done){
  console.log(res.value);
  res = g.next();
}
```

上面代码中，Generator 函数gen会自动执行完所有步骤。

但是，这不适合异步操作。如果必须保证前一步执行完，才能执行后一步，上面的自动执行就不可行。这时，Thunk 函数就能派上用处。以读取文件为例。下面的 Generator 函数封装了两个异步操作。

```
var fs = require('fs');
var thunkify = require('thunkify');
var readFileThunk = thunkify(fs.readFile);

var gen = function* (){
  var r1 = yield readFileThunk('/etc/fstab');
  console.log(r1.toString());
  var r2 = yield readFileThunk('/etc/shells');
  console.log(r2.toString());
};
```

上面代码中，yield命令用于将程序的执行权移出 Generator 函数，那么就需要一种方法，将执行权再交还给 Generator 函数。

这种方法就是 Thunk 函数，因为它可以在回调函数里，将执行权交还给 Generator 函数。为了便于理解，我们先看如何手动执行上面这个 Generator 函数。

```
var g = gen();

var r1 = g.next();
r1.value(function (err, data) {
  if (err) throw err;
  var r2 = g.next(data);
  r2.value(function (err, data) {
    if (err) throw err;
    g.next(data);
  });
});
```

上面代码中，变量g是 Generator 函数的内部指针，表示目前执行到哪一步。next方法负责将指针移动到下一步，并返回该步的信息（value属性和done属性）。

仔细查看上面的代码，可以发现 Generator 函数的执行过程，其实是将同一个回调函数，反复传入next方法的value属性。这使得我们可以用递归来自动完成这个过程。

**以下是基于 Promise 对象的自动执行**

```
var fs = require('fs');

var readFile = function (fileName){
  return new Promise(function (resolve, reject){
    fs.readFile(fileName, function(error, data){
      if (error) return reject(error);
      resolve(data);
    });
  });
};

var gen = function* (){
  var f1 = yield readFile('/etc/fstab');
  var f2 = yield readFile('/etc/shells');
  console.log(f1.toString());
  console.log(f2.toString());
};

// 手动执行： 利用 then 方法 层层添加回调函数
var g = gen();

g.next().value.then(function(data){
  g.next(data).value.then(function(data){
    g.next(data);
  });
});

// 自动执行器

function run(gen){
  var g = gen();

  function next(data){
    var result = g.next(data);
    if (result.done) return result.value;
    result.value.then(function(data){
      next(data);
    });
  }

  next();
}

run(gen);
```

+ 可以看到利用递归这一点，很容易在上面实现了 Generator 的自动执行器。

***Generator 是个很强大的东西，让异步程序可以写成同步的方式。此外，ES6 还实现了 async 和 await 两个关键字，其中自带执行器，其实就是 Generator 和自动执行器的语法糖。***



