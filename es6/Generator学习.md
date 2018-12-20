# Generator 的语法

>ES6 中为了更方便地写异步代码，在标准中引入了 `Async` 和 `Await`。而这么高大上的东西，其实就是 `Generator` 函数的语法糖。那么 Generator 是什么？

1. 基本概念

`Generator` 函数和 `Promise` 一样，也是ES6提供的一种异步编程的解决方案，语法行为与传统的函数完全不同。

**在语法上，可以把它理解成一个状态机，封装了多个内部状态。与其他函数不同的是，执行 Generator 函数不会返回结果，而是会返回一个遍历器，就是说，Generator除了是一个状态机，还是一个遍历器对象生成函数。返回的这个遍历器对象，可以依次遍历 Generator 函数内部的每一个状态。**

**在形式上，比普通函数多了两个特征。一是，`function` 关键字与函数名之间有一个星号；二是，函数体内部使用 `yield` 表达式，定义不同的内部状态。**

***举个栗子***
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
+ 上面定义的 Generator 函数内部有两个 yield 表达式和一个 return,即该函数有三个状态：hello， world和 return；和普通函数一样调用该函数，但并不执行，而是返回一个指向内部状态的指针对象，也就是遍历器对象。

+ 然后必须调用遍历器对象的 `next` 方法，使得指针移向下一个状态。每次调用 next 方法，内部指针就从函数头部或者上一次停下来的地方开始执行，直到遇到下一个 yield 表达式（或者return语句）为止。所以 Generator 函数是分段执行的，yield 表达式是暂停执行的标记，而 next 方法可以恢复执行。可以看到上面调用了四次 next 方法以及每次返回的结果。

+ 总结一下，调用 Generator 函数，返回一个遍历器对象，是一个指向 Generator 内部状态的指针。 之后调用该指针对象的 next 方法就会返回一个有着 `value` 和 `done` 两个属性的对象。 value 属性表示当前的内部状态的值，是 yield 表达式后面那个表达式的值，done 属性是一个布尔值，表示是否遍历结束。

2. `yield` 表达式

遍历器对象的 next 方法的运行逻辑如下：

***遇到 yield 表达式，就暂停执行后面的操作，并将紧靠在 yield 后面的那个表达式的值，作为返回的对象的 value 的属性值。***

***下一次调用 next 方法时，再继续往下执行，直到遇到下一个 yield 表达式。***

***如果没有遇到新的 yield 表达式，就一直运行到函数结束，或者遇到 return 语句为止，并将 return 语句后面的表达式的值作为返回对象的 value 的属性值。***

***如果该函数没有 return 语句，则返回对象的 value 属性值为 undefined ***

+ 注意：yield 后面的表达式不会立即求值，只会在 next 方法将指针移到这一句的时候才会被求值。

Generator 函数里面可以执行很多次 yield 和一次 return，所以 Generator 函数可以返回一系列值，状态。这就是生成器的由来。在 Generator 中如果执行到了 return，则就算后面还有 yield，该生成器也不在执行后面的 yield 了，遇到 return 后，返回的对象的done属性值为 true。

+ 另外需要注意，yield表达式只能用在 Generator 函数里面，用在其他地方都会报错

***举个栗子***
```
var arr = [1, [[2, 3], 4], [5, 6]];

var flat = function* (a) {
  a.forEach(function (item) {
    if (typeof item !== 'number') {
      yield* flat(item); // 这里会报错
    } else {
      yield item;
    }
  });
};

for (var f of flat(arr)){
  console.log(f);
}
```
上面这段代码是会报错的，虽然整体是在一个 Generator 函数里面，但是 forEach 接受的函数是 Generator 里面的另外一个函数，并不是 Generator 函数，所以在这个函数里面使用 yield 是会报错的，尽管它在一个 Generator 函数里面

2. 与 Iterator 接口的关系

在学习 Iterator 的时候知道任意一个对象的 `Symbol.iterator` 方法，等于该对象的遍历器生成函数，调用该函数会返回该对象的一个遍历器对象。而 Generator 函数就是一个遍历器生成函数，所以可以把 Generator 函数赋值给对象的 Symbol.iterator 属性，从而使得该对象具有 Iterator 接口。然后该对象就可以被 `...` 运算符 `for...of` 等遍历了。

***举个栗子***
```
var myIterable = {};
myIterable[Symbol.iterator] = function* () {
  yield 'Who';
  yield 'are';
  yield 'you';
};

[...myIterable] // ['Who', 'are', 'you']
```

***此外，Generator 函数执行后，返回一个遍历器对象。该对象本身也具有 Symbol.iterator 属性，执行后返回本身***
```
function* gen(){
  // some code
}

var g = gen();

g[Symbol.iterator]() === g
// true
```

3. next 方法的参数

yield 表达式本身是没有返回值的，或者说总是返回 undefined。next 方法可以带一个参数，该参数就会被当做**上一个** yield 表达式的返回值。

***举个栗子***
```
function* f() {
  for(var i = 0; true; i++) {
    var reset = yield i;
    if(reset) { i = -1; }
  }
}

var g = f();

g.next() // { value: 0, done: false }
g.next() // { value: 1, done: false }
g.next(true) // { value: 0, done: false }
```

+ 如果 next 方法没有参数，或者是 undefined，变量 reset 就总是 undefined，当 next 方法带一个参数 true 时，变量 reset 就被赋值为这个参数 true，因此进入判断把变量 i 赋值为 -1。

+ 这个功能有很重要的语法意义。Generator 函数从暂停状态到恢复运行，它的上下文状态（context）是不变的。通过next方法的参数，就有办法在 Generator 函数开始运行之后，继续向函数体内部注入值。也就是说，可以在 Generator 函数运行的不同阶段，从外部向内部注入不同的值，从而调整函数行为。

***另外一个栗子***
```
function* foo(x) {
  var y = 2 * (yield (x + 1));
  var z = yield (y / 3);
  return (x + y + z);
}

var a = foo(5);
a.next() // Object{value:6, done:false}
a.next() // Object{value:NaN, done:false}
a.next() // Object{value:NaN, done:true}

var b = foo(5);
b.next() // { value:6, done:false }
b.next(12) // { value:8, done:false }
b.next(13) // { value:42, done:true }
```

```
function* dataConsumer() {
  console.log('Started');
  console.log(`1. ${yield}`);
  console.log(`2. ${yield}`);
  return 'result';
}

let genObj = dataConsumer();
genObj.next();
// Started
genObj.next('a')
// 1. a
genObj.next('b')
// 2. b
```

+ 如果想要第一次调用next方法时，就能够输入值，可以在 Generator 函数外面再包一层

```
function wrapper(generatorFunction) {
  return function (...args) {
    let generatorObject = generatorFunction(...args);
    generatorObject.next(); // 这里调用了一次 next，让指针到 yield 处，下一次调用 next 就可以把参数传递到这里来。
    return generatorObject;
  };
}

const wrapped = wrapper(function* () {
  console.log(`First input: ${yield}`);
  return 'DONE';
});

wrapped().next('hello!')
// First input: hello!
```

4. Generator.prototype.throw()

Generator 函数返回的遍历器对象，都有一个throw方法，可以在函数体外抛出错误，**然后在 Generator 函数体内捕获**。

```
var g = function* () {
  try {
    yield;
  } catch (e) {
    console.log('内部捕获', e);
  }
};

var i = g();
i.next();

try {
  i.throw('a');
  i.throw('b');
} catch (e) {
  console.log('外部捕获', e);
}
// 内部捕获 a
// 外部捕获 b
```


+ 上面代码中，遍历器对象i连续抛出两个错误。第一个错误被 Generator 函数体内的catch语句捕获。i第二次抛出错误，由于 Generator 函数内部的catch语句已经执行过了，不会再捕捉到这个错误了，所以这个错误就被抛出了 Generator 函数体，被函数体外的catch语句捕获。

throw方法可以接受一个参数，该参数会被catch语句接收，建议抛出Error对象的实例。

注意，不要混淆遍历器对象的throw方法和全局的throw命令。上面代码的错误，是用遍历器对象的throw方法抛出的，而不是用throw命令抛出的。后者只能被函数体外的catch语句捕获。

```
var g = function* () {
  while (true) {
    try {
      yield;
    } catch (e) {
      if (e != 'a') throw e;
      console.log('内部捕获', e);
    }
  }
};

var i = g();
i.next();

try {
  throw new Error('a');
  throw new Error('b');
} catch (e) {
  console.log('外部捕获', e);
}
// 外部捕获 [Error: a]
```

+ 上面代码之所以只捕获了a，是因为函数体外的catch语句块，捕获了抛出的a错误以后，就不会再继续try代码块里面剩余的语句了。

如果 Generator 函数内部没有部署try...catch代码块，那么throw方法抛出的错误，将被外部try...catch代码块捕获。

如果 Generator 函数内部和外部，都没有部署try...catch代码块，那么程序将报错，直接中断执行。

```
var gen = function* gen(){
  yield console.log('hello');
  yield console.log('world');
}

var g = gen();
g.next();
g.throw();
// hello
// Uncaught undefined
```

+ throw方法抛出的错误要被内部捕获，前提是必须至少执行过一次next方法。

```
function* gen() {
  try {
    yield 1;
  } catch (e) {
    console.log('内部捕获');
  }
}

var g = gen();
g.throw(1);
// Uncaught 1
```

+ 上面代码中，g.throw(1)执行时，next方法一次都没有执行过。这时，抛出的错误不会被内部捕获，而是直接在外部抛出，导致程序出错。这种行为其实很好理解，因为第一次执行next方法，等同于启动执行 Generator 函数的内部代码，否则 Generator 函数还没有开始执行，这时throw方法抛错只可能抛出在函数外部。

***throw方法被捕获以后，会附带执行下一条yield表达式。也就是说，会附带执行一次next方法。***

一旦 Generator 执行过程中抛出错误，且没有被内部捕获，就不会再执行下去了。如果此后还调用next方法，将返回一个value属性等于undefined、done属性等于true的对象，即 JavaScript 引擎认为这个 Generator 已经运行结束了。

```
function* g() {
  yield 1;
  console.log('throwing an exception');
  throw new Error('generator broke!');
  yield 2;
  yield 3;
}

function log(generator) {
  var v;
  console.log('starting generator');
  try {
    v = generator.next();
    console.log('第一次运行next方法', v);
  } catch (err) {
    console.log('捕捉错误', v);
  }
  try {
    v = generator.next();
    console.log('第二次运行next方法', v);
  } catch (err) {
    console.log('捕捉错误', v);
  }
  try {
    v = generator.next();
    console.log('第三次运行next方法', v);
  } catch (err) {
    console.log('捕捉错误', v);
  }
  console.log('caller done');
}

log(g());
// starting generator
// 第一次运行next方法 { value: 1, done: false }
// throwing an exception
// 捕捉错误 { value: 1, done: false }
// 第三次运行next方法 { value: undefined, done: true }
// caller done
```

+ 上面代码一共三次运行next方法，第二次运行的时候会抛出错误，然后第三次运行的时候，Generator 函数就已经结束了，不再执行下去了。

5. Generator.prototype.return()

Generator 函数返回的遍历器对象，还有一个return方法，可以返回给定的值，并且**终结**遍历 Generator 函数。

```
function* gen() {
  yield 1;
  yield 2;
  yield 3;
}

var g = gen();

g.next()        // { value: 1, done: false }
g.return('foo') // { value: "foo", done: true }
g.next()        // { value: undefined, done: true }
```

+ 如果 Generator 函数内部有try...finally代码块，且正在执行try代码块，那么return方法会推迟到finally代码块执行完再执行。

```
function* numbers () {
  yield 1;
  try {
    yield 2;
    yield 3;
  } finally {
    yield 4;
    yield 5;
  }
  yield 6;
}
var g = numbers();
g.next() // { value: 1, done: false }
g.next() // { value: 2, done: false }
g.return(7) // { value: 4, done: false }
g.next() // { value: 5, done: false }
g.next() // { value: 7, done: true }
```

+ 上面代码中，调用return方法后，就开始执行finally代码块，然后等到finally代码块执行完，再执行return方法。

6. yield* 表达式

如果在 Generator 函数内部，调用另一个 Generator 函数，默认情况下是没有效果的。就需要用到yield*表达式，用来在一个 Generator 函数里面执行另一个 Generator 函数。

```
function* foo() {
  yield 'a';
  yield 'b';
}

function* bar() {
  yield 'x';
  yield* foo();
  yield 'y';
}

// 等同于
function* bar() {
  yield 'x';
  yield 'a';
  yield 'b';
  yield 'y';
}

// 等同于
function* bar() {
  yield 'x';
  for (let v of foo()) {
    yield v;
  }
  yield 'y';
}

for (let v of bar()){
  console.log(v);
}
// "x"
// "a"
// "b"
// "y"
```

+ 再来看一个对比的例子。

```
function* inner() {
  yield 'hello!';
}

function* outer1() {
  yield 'open';
  yield inner();
  yield 'close';
}

var gen = outer1()
gen.next().value // "open"
gen.next().value // 返回一个遍历器对象
gen.next().value // "close"

function* outer2() {
  yield 'open'
  yield* inner()
  yield 'close'
}

var gen = outer2()
gen.next().value // "open"
gen.next().value // "hello!"
gen.next().value // "close"
```

+ 上面例子中，outer2使用了yield*，outer1没使用。结果就是，outer1返回一个遍历器对象，outer2返回该遍历器对象的内部值。从语法角度看，如果yield表达式后面跟的是一个遍历器对象，需要在yield表达式后面加上星号，表明它返回的是一个遍历器对象。这被称为yield*表达式。

7. 作为对象属性的 Generator 函数

如果一个对象的属性是 Generator 函数，可以简写成下面的形式。

```
let obj = {
  * myGeneratorMethod() {
    ···
  }
};

// 等价于

let obj = {
  myGeneratorMethod: function* () {
    // ···
  }
};
```

8. Generator 函数的 this

Generator 函数总是返回一个遍历器，ES6 规定这个遍历器是 Generator 函数的实例，也继承了 Generator 函数的prototype对象上的方法。

```
function* g() {}

g.prototype.hello = function () {
  return 'hi!';
};

let obj = g();

obj instanceof g // true
obj.hello() // 'hi!'
```

+ 上面代码表明，Generator 函数g返回的遍历器obj，是g的实例，而且继承了g.prototype。但是，如果把g当作普通的构造函数，并不会生效，因为g返回的总是遍历器对象，而不是this对象。

***复杂些的栗子***
```
function* gen() {
  this.a = 1;
  yield this.b = 2;
  yield this.c = 3;
}

function F() {
  return gen.call(gen.prototype);
}

var f = new F();

f.next();  // Object {value: 2, done: false}
f.next();  // Object {value: 3, done: false}
f.next();  // Object {value: undefined, done: true}

f.a // 1
f.b // 2
f.c // 3
```

***这里先讲到这里，关于 Generator 的异步应用，请移步***
