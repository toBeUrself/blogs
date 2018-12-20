# Generator 的语法以及异步应用

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

