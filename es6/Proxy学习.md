> Proxy 用于修改某些操作的默认行为，等同于在语言层面做出修改，所以属于一种'元编程'(meta programming),即对编程语言进行编程。

1. 概述

****

Proxy 可以理解成，在目标之间架设一层'拦截'，外界对该对象的访问，都必须先通过这层拦截，因此提供了一种机制，可以对外界的访问进行过滤和改。Proxy 这个词的原意就是代理，用在这里表示由它来 代理 某些操作，可以理解为'代理器'。

```
var obj = new Proxy({}, {
  get: function (target, key, receiver) {
    console.log(`getting ${key}!`);
    return Reflect.get(target, key, receiver);
  },
  set: function (target, key, value, receiver) {
    console.log(`setting ${key}!`);
    return Reflect.set(target, key, value, receiver);
  }
});
```

+ 上面代码对空对象架设了一层拦截，重定义了属性的读取 `get` 和 设置 `set` 行为。这里暂时先不解释具体的语法，只看结果。对设置了拦截行为的对象 `obj`，去读写它的属性，就会得到下面的结果。

```
obj.count = 1
//  setting count!
++obj.count
//  getting count!
//  setting count!
//  2
```

+ 上面代码说明，Proxy 实际上重载了点运算符，即用自己的定义覆盖了语言的原始定义。

ES6 原生提供 Proxy 构造函数，用来生成 Proxy 实例。

```
let proxy = new Proxy(target, handler);
```

+ Proxy 对象的所有用法，都是上面这种形式，不同的只是 `handler` 参数的写法。其中，`new Proxy()` 表示生成一个 Proxy 实例，`target` 参数表示所要拦截的目标对象，`handler` 参数也是一个对象，用来控制拦截行为。

下面是另一个拦截读取属性行为的例子

```
var proxy = new Proxy({}, {
  get: function(target, property) {
    return 35;
  }
});

proxy.time // 35
proxy.name // 35
proxy.title // 35
```

上面代码中，作为构造函数，`Proxy` 接受两个参数。第一个参数是所要代理的目标对象，第二个参数是一个配置对象，对于每一个被代理的操作，需要提供一个对应的处理函数，该函数将拦截对应的操作。比如，上面代码中，配置对象有一个 get 方法，用来拦截对目标对象属性的访问请求。
`get` 方法的两个参数分别是目标对象和所要访问的属性。可以看到，由于拦截函数总是返回 35，所以访问任何属性都得到 35。

+ 注意，想要 Proxy 起作用，必须针对 Proxy 实例(上例是`proxy`对象)进行操作，而不是针对目标(上例是空对象)进行操作。

如果 `handler` 没有设置任何拦截，那就等同于直接通向原对象。

```
var target = {};
var handler = {};
var proxy = new Proxy(target, handler);
proxy.a = 'b';
target.a // "b"
```

+ 有一个技巧: 将 Proxy 对象设置到 object.proxy 属性，从而可以在 object 对象上调用。

```
var object = { proxy: new Proxy(target, handler) };
```

Proxy 实例也可以作为其他对象的原型对象。

```
var proxy = new Proxy({}, {
  get: function(target, property) {
    return 35;
  }
});

let obj = Object.create(proxy);
obj.time // 35
```

上面代码中，proxy 对象是 obj 对象的原型，obj 对象本身并没有 time 属性，所以根据原型链，会在 proxy 对象上读取该属性，导致被拦截。





