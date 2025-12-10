---
title: js手写系列————手写call、apply、bind方法
date: 2021-05-12 13:26:26
updated: 2021-05-12 13:26:26
tags: [js]
categories: js手写系列
description: call，apply，bind是JavaScript中用于改变普通函数this指向（无法改变箭头函数this指向）的方法
---
# 什么是call、bind、apply？
> call，apply，bind是JavaScript中用于改变普通函数this指向（无法改变箭头函数this指向）的方法，这三个函数实际上都是绑定在Function构造函数的prototype上，而每一个函数都是Function的实例，因此每一个函数都可以直接调用call，apply，bind

# call、bind、apply的使用
## call方法
- 语法： function.call(thisArg, arg1, arg2, ...)。 其中thisArg是要设置为函数执行上下文的对象，也就是this要指向的对象，从第二个参数开始，arg1, arg2, ... 是传递给函数的参数。通过使用call方法，可以将一个对象的方法应用到另一个对象上。
```js
// 定义一个对象
const person1 = {
  name: 'Alice',
  greet: function() {
    console.log(`Hello, ${this.name}!`);
  }
};

// 定义另一个对象
const person2 = {
  name: 'Bob'
};

// 使用call方法将person1的greet方法应用到person2上
person1.greet.call(person2); // 输出：Hello, Bob!
```

## apply方法
- 语法：function.apply(thisArg, [argsArray])。 其中thisArg是要设置为函数执行上下文的对象，也就是this要指向的对象，argsArray是一个包含参数的数组。通过使用apply方法，可以将一个对象的方法应用到另一个对象上，并使用数组作为参数。
```js
function greet(name) {
  console.log(`Hello, ${name}!`);
}

const person = { name: 'John' };
greet.apply(person, ['Mary']); // 输出：Hello, Mary!
```

## bind方法
- 语法：function.bind(thisArg, arg1, arg2, ...)。 其中thisArg是要绑定到函数执行上下文的对象，也就是this要指向的对象，从第二个参数开始，arg1, arg2, ...是传递给函数的参数。与call和apply方法不同，bind方法并不会立即执行函数，而是返回一个新函数，可以稍后调用。这对于事件处理程序和setTimeout函数等场景非常有用。
```js
function greet(name) {
  console.log("Hello, " + name);
}

const delayedGreet = greet.bind(null, "John");
setTimeout(delayedGreet, 2000);  // 2秒后输出：Hello, John
```

# call、bind、apply的区别
1. 调用方式：
   - call：使用函数的call方法可以直接调用函数，并传递参数列表。
   - apply：使用函数的apply方法可以直接调用函数，并传递参数列表，与call方法类似，但参数需要以数组或类数组的形式传递。
   - bind：使用函数的bind方法可以返回一个新的函数，这个新函数的this值被绑定到指定的对象，但不会立即执行。

2. 参数传递方式：
   - call：使用call方法时，参数需要一个一个地列举出来，通过逗号分隔。
   - apply：使用apply方法时，参数需要以数组或类数组的形式传递。
   - bind：使用bind方法时，可以传递任意数量的参数，可以在绑定时传递参数，也可以在调用时传递参数。

3. 执行时机：
   - call：调用call方法时，函数会立即执行。
   - apply：调用apply方法时，函数会立即执行。
   - bind：调用bind方法时，返回一个新函数，需要后续再调用这个新函数才会执行。

# 手写call、apply、bind方法
## 手写call()
- 原理：
  1. 首先，通过 **Function.prototype.myCall** 将自定义的 **myCall** 方法添加到所有函数的原型对象上，使得所有函数实例都可以调用该方法。
  2. 在 myCall 方法内部，首先通过 **typeof this !== "function"** 判断调用 **myCall** 的对象是否为函数。如果不是函数，则抛出一个类型错误。
  3. 然后，判断是否传入了上下文对象 **context**。如果没有传入，则将 **context** 赋值为全局对象；ES11 引入了 **globalThis**，它是一个统一的全局对象，无论在浏览器还是 **Node.js** 中，都可以使用 **globalThis** 来访问全局对象, 如果传入的不是对象时，**context** 是一个对应的包装类型。
  4. 接下来，使用 **Symbol** 创建一个唯一的键 **key**，用于将调用 **myCall** 的函数绑定到上下文对象的新属性上。
  5. 将调用 **myCall** 的函数赋值给上下文对象的 **key** 属性，实现了将函数绑定到上下文对象上的效果，使用 **Object.defineProperty** 可以做到使其不被枚举出来。
  6. 调用绑定在上下文对象上的函数，并传入 **myCall** 方法的其他参数 **args**。
  7. 将绑定在上下文对象上的函数删除，以避免对上下文对象造成影响。
  8. 返回函数调用的结果。
```js
   Function.prototype.myCall = function (context, ...args) {
  // 判断调用myCall的是否为函数
  if (typeof this !== "function") {
    throw new TypeError("Function.prototype.myCall - 被调用的对象必须是函数");
  }

  // 如果没有传入上下文对象，则默认为全局对象
  // ES11 引入了 globalThis，它是一个统一的全局对象
  // 无论在浏览器还是 Node.js 中，都可以使用 globalThis 来访问全局对象。
  context = context === null || context === undefined ?  globalThis : Object(context);

  // 用Symbol来创建唯一的key，防止名字冲突
  const key = Symbol();
  const fn  = this;
  // this是调用myCall的函数，将函数绑定到上下文对象的新属性上
  Object.defineProperty(context, key, {
    value: fn,
    configurable: true  
    // 需要注意的是，使用 Object.defineProperty() 方法定义的属性默认情况下是不可配置的，这意味着不能使用 delete 关键字删除这些属性。如果需要定义可配置的属性，可以在 descriptor 对象中将 configurable 属性设置为 true。
  })
  // context[key] = this; 这种方式会被枚举出来

  // 传入MyCall的多个参数
  const result = context[key](...args);

  // 将增加的fn方法删除
  delete context[key];

  return result;
};
```

- 测试
```js
const test = {
  name: "xxx",
  hello: function () {
    console.log(`hello,${this.name}!`);
  },
  add: function (a, b) {
    return a + b;
  },
};
const obj = { name: "world" };
test.hello.myCall(obj); //hello,world!
test.hello.call(obj);//hello,world!
console.log(test.add.myCall(null, 1, 2));//3
console.log(test.add.call(null, 1, 2));//3
```

## 手写apply()
- 原理：apply的实现思路跟call类似，就是apply传入参数是以数组的形式传入，所以多了一步判断传入的参数是否为数组以及在调用方法的时候使用扩展运算符 ... 将传入的参数数组 argsArr 展开
```js
Function.prototype.myApply = function (context, argsArr) {
  // 判断调用myApply的是否为函数
  if (typeof this !== "function") {
    throw new TypeError("Function.prototype.myApply - 被调用的对象必须是函数");
  }

  // 判断传入的参数是否为数组
  if (argsArr && !Array.isArray(argsArr)) {
    throw new TypeError("Function.prototype.myApply - 第二个参数必须是数组");
  }

  // 如果没有传入上下文对象，则默认为全局对象
  // ES11 引入了 globalThis，它是一个统一的全局对象
  // 无论在浏览器还是 Node.js 中，都可以使用 globalThis 来访问全局对象。
  context = context === null || context === undefined ?  globalThis : Object(context);
  
  //如果第二个参数省略则赋值空数组
  argsArr = argsArr || [];

  // 用Symbol来创建唯一的key，防止名字冲突
  const key = Symbol();
  const key = this;
  // this是调用myApply的函数，将函数绑定到上下文对象的新属性上
  Object.defineProperty(context, key, {
    value: fn,
    configurable: true  
  })

  // 传入myApply的多个参数
  const result = context[key](...argsArr)

  // 将增加的fn方法删除
  delete context[key];

  return result;
};
```

- 测试
```js
const test = {
  name: "xxx",
  hello: function () {
    console.log(`hello,${this.name}!`);
  },
};
const obj = { name: "world" };
test.hello.myApply(obj); //hello,world!
test.hello.apply(obj); //hello,world!
const arr = [2,3,6,5,1,7,9,5,0]
console.log(Math.max.myApply(null,arr));//9
console.log(Math.max.apply(null,arr));//9
```

## 手写bind()
- 原理：
  1. 首先，通过 **Function.prototype.myBind** 将自定义的 **myBind** 方法添加到所有函数的原型对象上，使得所有函数实例都可以调用该方法。
  2. 在 **myBind** 方法内部，首先通过 **typeof this !== "function"** 判断调用 **myBind** 的对象是否为函数。如果不是函数，则抛出一个类型错误。
  3. 然后，判断是否传入了上下文对象 **context**。如果没有传入，则将 **context** 赋值为全局对象；ES11 引入了 **globalThis**，它是一个统一的全局对象，无论在浏览器还是 **Node.js** 中，都可以使用 **globalThis** 来访问全局对象。
  4. 保存原始函数的引用，使用 **_this** 变量来表示。
  5. 返回一个新的闭包函数 **fn** 作为绑定函数。这个函数接受任意数量的参数 **innerArgs**。（关于闭包的介绍可以看这篇文章->闭包的应用场景）
  6. 在返回的函数 **fn** 中，首先判断是否通过 **new** 关键字调用了函数。这里需要注意一点，如果返回出去的函数被当作构造函数使用，即使用 **new** 关键字调用时，**this** 的值会指向新创建的实例对象。通过检查 **this instanceof fn**，可以判断返回出去的函数是否被作为构造函数调用。这里使用 **new _this(...args, ...innerArgs)** 来创建新对象。
  7. 如果不是通过 **new** 调用的，就使用 **apply** 方法将原始函数 **_this** 绑定到指定的上下文对象 **context** 上。这里使用 **apply** 方法的目的是将参数数组 **args.concat(innerArgs)** 作为参数传递给原始函数
```js
Function.prototype.myBind = function (context, ...args) {
  // 判断调用myBind的是否为函数
  if (typeof this !== "function") {
    throw new TypeError("Function.prototype.myBind - 被调用的对象必须是函数");
  }

  // 如果没有传入上下文对象，则默认为全局对象
  // ES11 引入了 globalThis，它是一个统一的全局对象
  // 无论在浏览器还是 Node.js 中，都可以使用 globalThis 来访问全局对象。
  context = context || globalThis;

  // 保存原始函数的引用，this就是要绑定的函数
  const _this = this;

  // 返回一个新的函数作为绑定函数
  return function fn(...innerArgs) {
    // 判断返回出去的函数有没有被new
    // if(new.target) {  es6判断函数有没有被new
    if (this instanceof fn) {
      return new _this(...args, ...innerArgs);
    }
    // 使用apply方法将原函数绑定到指定的上下文对象上
    return _this.apply(context,args.concat(innerArgs));
  };
};
```

- 测试
```js
const test = {
  name: "xxx",
  hello: function (a,b,c) {
    console.log(`hello,${this.name}!`,a+b+c);
  },
};
const obj = { name: "world" };
let hello1 = test.hello.myBind(obj,1);
let hello2 = test.hello.bind(obj,1); 
hello1(2,3)//hello,world! 6
hello2(2,3)//hello,world! 6
console.log(new hello1(2,3));
//hello,undefined! 6
// hello {}
console.log(new hello2(2,3));
//hello,undefined! 6
// hello {}
```

# 总结
在JavaScript中，call、bind 和 apply 方法是非常有用的工具，我们可以使用call、bind、apply方法来改变函数执行中的this指向，帮助我们更好地控制函数的执行上下文。在实际开发中，这些方法都有各自的应用场景，需要根据具体情况选择使用。