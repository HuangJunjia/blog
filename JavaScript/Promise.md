## Promise学习

-----

[TOC]

-----

##### 一、回调函数的问题

学过 JavaScript 的人都应该清楚一件事，JavaScript语言的一大特点就是`单线程`，目的是为了提高效率。因此在 JavaScript 中回调函数不会立即执行，而是由事件轮询去检测事件是否执行完毕，当执行完毕并且有结果之后，将执行结果放入回调函数中，然后将回调函数添加到事件队列中等待被执行。

在这里会有一些关于回调函数的问题：

1. “回调地狱”：即“洋葱模型”，回调函数作为异步函数的参数，会形成多级的嵌套，当嵌套级数过多时，代码逻辑会变得混乱，无法将错误的捕捉和处理这个简单的工作做好，只能在回调函数的内部通过`try{}...catch(){}`捕获并处理异常。
2. 回调函数的执行方式不符合自然语言的线性思维方式，不易理解。
3. 控制反转，即控制权不在我们手中，而是在其他人的代码中。例如该异步函数是第三方库，当我们把回调函数传给第三方库的时候，我们并不能知道我们的异步函数在第三方库里做了什么，在调用回调函数之前做了什么。

解决回调函数的问题有很多种方法，其中比较好的一种方式就是使用 Promise 对象。

##### 二、什么是Promise

Promise 是目前在 JavaScript 异步编程中比较流行的解决方案之一，Promise 用于表示一个异步操作的最终状态（完成或者失败），并且可以链式的处理异步请求（`.then()`方法），很好的处理异常问题，是解决回调地狱的良好方案之一。MDN对 Promise 的[解释](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise#%E6%8F%8F%E8%BF%B0)如下：

> `Promise` 对象是一个代理对象（代理一个值），被代理的值在 `Promise `对象创建时可能是未知的。它允许你为异步操作的成功和失败分别绑定相应的处理方法（handlers）。 这让异步方法可以像同步方法那样返回值，但并不是立即返回最终执行结果，而是一个能代表未来出现的结果的 `promise `对象。

##### 三、Promise 的状态

Promise 一共包含有三种状态：

1. `pending`：初始状态，既不是成功状态，也不是失败状态，在网上多数人称为`等待中`状态。
2. `fulfilled`：成功状态，意味着操作成功完成。
3. `rejected`：失败状态，意味着操作失败。

Promise 无论如何都会返回一个结果，不是成功，就是失败。并且 Promise 的设计具有原子性，即当状态从`pending`状态转变为`fulfilled`状态或者`rejected`状态后，将不能改变。

在`pending`状态中，Promise 可能触发`fulfilled`状态并将成功结果传递给相应的状态处理方法，也可能触发`rejected`状态并将失败信息返回。

![图源自[@MDN的Promise文档](https://mdn.mozillademos.org/files/8633/promises.png)](./imgs/promises.png)

##### 四、Promise 的原理

从本文前面的内容中可知 Promise 有三个状态，并且会返回一个成功或者失败的结果。

![图源来自[《JS异步编程之Promise》](https://juejin.im/post/5c724f14518825626b76f6d5)](./imgs/0.jpg)

(图来自于https://juejin.im/post/5c724f14518825626b76f6d5)

``` javascript
var promise = new Promise((resolve, reject) => {
    console.log("Promise构造器执行");
    setTimeout(() => {
        if (true) {
            resolve("成功");
        } else {
            reject("失败");
        }
    }, 1000)
})
promise.then((result) => {
    console.log(result);
    return 1;
    
    // return Promise.resolve(1);  // 返回一个决议为成功的 Promise 实例
    // return Promise.reject("error");  // 返回一个决议为拒绝的 Promise 实例
})
.then((result) => {
    // .then() 方法会返回一个 Promise, 完成调用的参数为前一个 Promise 的返回值或者决议值。
    console.log(result);
    throw new Error("抛出错误"); // 抛出错误是隐式拒绝
})
.catch((error) => {
    console.log(error);
})
.then(() => {
    // Continue to do other things
})
.finally(() => {
    console.log("finally");
})
```

##### 五、Promise 的语法

``` javascript
new Promise( function(resolve, reject) {...} /* executor */  );
```

参数：

`executor`：

- `executor`函数带有`resolve`和`reject`两个参数。当 Promise 构造函数在执行时会**立即调用**`executor`函数，`executor`函数在 Promise 构造函数返回新建对象之前会被调用，并将`resolve`和`reject`两个函数作为参数传递给`executor`函数。在`executor`函数内部通常会执行一些异步操作，并在异步操作完成时调用`resolve`函数或者`reject`函数将 promise 的状态修改为`fulfilled(成功)`或者`rejected(失败)`
- 只要在`executor`函数中抛出一个错误，promise 的状态就会转换为`rejected`，此时`executor`函数的返回值将会被忽略。

##### 六、Promise 的属性

* `Promise.length`：

   Promise 的 length 属性，其值始终为 1，即构造器参数的数目。

* `Promise.prototype`：

   Promise 构造器的原型。

##### 七、Promise 的方法

* `Promise.all(iterable)`：

  当 promise 对象中 iterable 参数对象里所有的 promise 对象都成功执行的时候触发的一个方法，若 iterable 中有任何一个 promise 对象执行失败则会立即触发该 promise 对象的失败。该方法在触发成功状态之后，会将 iterable 参数里所有 promise 对象返回值放入一个数组中并将该数组作为成功回调的返回值，该数组中各个 promise 对象的返回值顺序与 iterable 的顺序保持一致。如果触发了失败状态，则该方法会将第一个触发失败状态的 promise 对象的错误信息作为它的错误信息返回。该方法常用于处理多个 promise 对象的状态合集。

* `Promise.race(iterable)`：

  iterable 参数中只要有一个 promise 对象触发了额成功状态或者失败状态，就会将该 promise 对象的值作为它的返回值。

* `Promise.reject(reason)`：

  返回一个状态为失败的 Promise 对象，并将给定的失败信息传递给对应的处理方法。

* `Promise.resolve(value)`：

> 返回一个状态由给定 value 决定的 Promise 对象。如果该值是一个 Promise 对象，则直接返回该对象；如果该值是 thenable (即，带有`.then()`方法的对象)，返回的 Promise 对象的最终状态由`.then()`方法执行决定；否则的话(该 value 为空，基本类型或者不带`.then()`方法的对象)，返回的 Promise 对象状态为`fulfilled`，并且将该 value 传递给对应的`.then()`方法。通常而言，如果你不知道一个值是否是 Promise 对象，使用`Promise.resolve(value)` 来返回一个 Promise 对象,这样就能将该 value 以 Promise 对象形式使用。

##### 八、Promise 的原型

* 属性：

  * `Promise.prototype.construtor`：

    返回被创建的实例函数.  默认为 Promise 函数。

* 方法：

  * `Promise.prototype.catch(onRejected)`：

    当`.then()`方法中发生错误时，`.catch()`方法会捕获并处理错误，并将一个`rejection(拒绝)`回调到当前的 promise，然后返回一个新的 promise。新的 promise 以`.catch()`的返回值来 resolve。

  * `Promise.prototype.then(onFulfilled, onRejected)`：

    在当前的 promise 中添加`fulfilled(解决)`回调和`rejuection(拒绝)`回调，并以回调的返回值来 resolve。

  * `Promise.prototype.finally(onFinally)`：

    无论 promise 的状态是处于`fulfilled(成功)`状态还是`rejected(失败)`状态，都会调用的一个方法，并且在该回调中返回一个新的 promise 对象。

##### 九、Promise 的优势

1. 链式调用：

   Promise 在使用后会返回一个新的 Promise 便于我们传递状态参数。同时因为其链式的写法更接近于同步写法，更加符合线性思维。

2. 错误捕捉：

   Promise 能够为链式异步调用提供错误处理。

3. 控制权的再次反转：

   第三方提供的异步函数我们无法保证回调函数如何被执行，但是通过 Promise 我们能够保证 resolve 只会执行一次，并且 Promise 始终以异步的形式执行。

4. 解决未决议和并行嵌套的问题：

   Promise 的`Promise.all(iterable)`方法和`Promise.race(iterable)`方法可以用于解决 Promise 始终未决议和并行 Promise 嵌套的问题。

##### 十、Promise对象的不足

1. 每个`.then()`方法都是一个独立的作用域：

   当我们加入多个`.then()`方法时，会创建多个独立的作用域，要想解决作用域的数据共享问题需要在外层包裹一层函数作用域实现闭包。

2. `.then()`无法取消：

   `.catch()`能捕获并处理 Promise 链中任意一个`.then()`方法中的错误，但是不会中断整个 Promise 链的执行。

3. 无法得知进度：

   Promise 只会从`pending`状态转变为`fullfilled`状态或者`rejected`状态，所以我们无法得知`pending`阶段的进度。

##### 十一、应用 Promise 的简单例子

``` javascript
// 使用 Promise 对ajax进行封装
function fetch(method, url, data){
    return new Promise((resolve, reject) => {
        var xhr = new XMLHttpRequest();
        var method = method || "GET";
        var data = data || null;
        xhr.open(method, url, true);
        xhr.onreadystatechange = function() {
            if(xhr.status === 200 && xhr.readyState === 4){
                resolve(xhr.responseText);
            } else {
                reject(xhr.responseText);
            }
        }
        xhr.send(data);
        })
}

// 使用
fetch("GET", "/api", null)
.then(result => {
    console.log(result);
})

// 封装 nodejs error first 风格回调
function readFile(url) {
    return new Promise((resolve, reject) => {
       fs.readFile(url,'utf8', (err, data) => {
        if(err) {
            reject(err);
            return;
        }
        resolve(data)
        }) 
    })
}
```

-----

###### 参考资料（排名不分先后顺序）

1. [@南波的《JS异步编程之Promise》](https://juejin.im/post/5c724f14518825626b76f6d5)
2. [@南波的《JS异步编程之callback》](https://juejin.im/post/5c691ef851882562c0496759)
3. [@蟹丸的《前端异步技术之Promise》](https://juejin.im/post/5c71927c6fb9a049cd54d2de)
4. [MDN的Promise文档](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise)

