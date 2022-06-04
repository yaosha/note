### Promise

Promise用于异步编程。
一个 Promise 必然处于以下几种状态之一：

* pending：初始状态
* fulfilled：成功
* rejected：失败

```js
// 成功使用resolve将状态改为fulfilled，失败使用reject将状态改为rejected
const promise = new Promise(function(resolve, reject) {
  // 异步操作
  if (异步操作成功) {
    resolve(value);
  } else {
    reject(error);
  }
});

// Promise的返回值也是Promise对象，可以链式调用
promise.then(function(value) {
  // 成功才会执行，value是上一步中resolve的参数
}, function(error) {
  // 失败才会执行，error是上一步中reject的参数
});

```

```js
let promise = new Promise(function(resolve, reject) {
  resolve(1);
});
promise = promise.then(val => {
  console.log(val);
  return 2;
}).then(val => {
  console.log(val);
  return 3;
});
// 1
// 2

promise.then(val => {
  throw new Error('error');
  return 3;
}).catch(error => {
  console.log(error);
}).finally(() => {
  console.log('finally');
});
// Error: error
// finally
```

### Promise.resolve()

返回一个fulfilled的Promise

```js
const promise1 = Promise.resolve(123);

promise1.then((value) => {
  console.log(value);
  // expected output: 123
});

```

### Promise.reject()

返回一个带有拒绝原因的 Promise 对象。

```js
function resolved(result) {
  console.log('Resolved');
}

function rejected(result) {
  console.error(result);
}

Promise.reject(new Error('fail')).then(resolved, rejected);
// expected output: Error: fail
```

### Promise.all()

参数：Promise的iterable 类型
返回值：一个Promise实例，在所有输入的promise的resolve执行完的时候执行实例的resolve，在任何一个输入的 promise 的 reject 回调执行的时候执行实例的reject

```js
const promise1 = Promise.resolve(3);
const promise2 = 42;
const promise3 = new Promise((resolve, reject) => {
  setTimeout(resolve, 100, 'foo');
});

Promise.all([promise1, promise2, promise3]).then((values) => {
  console.log(values);
});
// expected output: Array [3, 42, "foo"]
```

失败
```js
var p1 = new Promise((resolve, reject) => {
  setTimeout(resolve, 1000, 'one');
});
var p2 = new Promise((resolve, reject) => {
  setTimeout(resolve, 2000, 'two');
});
var p3 = new Promise((resolve, reject) => {
  setTimeout(resolve, 3000, 'three');
});
var p4 = new Promise((resolve, reject) => {
  setTimeout(resolve, 4000, 'four');
});
var p5 = new Promise((resolve, reject) => {
  reject('reject');
});

Promise.all([p1, p2, p3, p4, p5]).then(values => {
  console.log(values);
}, reason => {
  console.log(reason) // "reject"
});



// 也可以使用catch
Promise.all([p1, p2, p3, p4, p5]).then(values => {
  console.log(values);
}).catch(reason => {
  console.log(reason) // "reject"
});

```

### Promise.allSettled()

allSettled的参数和all一样，区别在于所有输入的promise的resolve和reject回调执行完毕后才会执行返回值的resolve

```js
const promise1 = Promise.resolve(3);
const promise2 = new Promise((resolve, reject) => setTimeout(reject, 100, 'foo'));
const promises = [promise1, promise2];

Promise.allSettled(promises).
  then((results) => results.forEach((result) => console.log(result)));
```
// {status: 'fulfilled', value: 3}
// {status: 'rejected', reason: 'foo'}

### Promise.any()

Promise.any()接收一个Promise可迭代对象，和 Promise.all() 是相反的。
只要其中的一个 promise 成功，就返回那个已经成功的 promise ；如果可迭代对象中所有的 promises 都失败，就返回一个失败的 promise。

```js
const pErr = new Promise((resolve, reject) => {
  reject("总是失败");
});

const pSlow = new Promise((resolve, reject) => {
  setTimeout(resolve, 500, "最终完成");
});

const pFast = new Promise((resolve, reject) => {
  setTimeout(resolve, 100, "很快完成");
});

Promise.any([pErr, pSlow, pFast]).then((value) => {
  console.log(value);
})
// 很快完成

const pErr = new Promise((resolve, reject) => {
  reject('总是失败');
});

Promise.any([pErr]).catch((err) => {
  console.log(err);
})
// AggregateError: All promises were rejected
```

### Promise.race()

Promise.race(iterable) 方法接收一个Promise可迭代对象，返回一个 promise，一旦迭代器中的某个 promise 成功或失败，返回的 promise 就会成功或失败。

```js
var p1 = new Promise(function(resolve, reject) {
    setTimeout(resolve, 500, "one");
});
var p2 = new Promise(function(resolve, reject) {
    setTimeout(resolve, 100, "two");
});

Promise.race([p1, p2]).then(function(value) {
  console.log(value); // "two"
  // 两个都完成，但 p2 更快
});

var p3 = new Promise(function(resolve, reject) {
    setTimeout(resolve, 100, "three");
});
var p4 = new Promise(function(resolve, reject) {
    setTimeout(reject, 500, "four");
});

Promise.race([p3, p4]).then(function(value) {
  console.log(value); // "three"
  // p3 更快，所以它完成了
}, function(reason) {
  // 未被调用
});

var p5 = new Promise(function(resolve, reject) {
    setTimeout(resolve, 500, "five");
});
var p6 = new Promise(function(resolve, reject) {
    setTimeout(reject, 100, "six");
});

Promise.race([p5, p6]).then(function(value) {
  // 未被调用
}, function(reason) {
  console.log(reason); // "six"
  // p6 更快，所以它失败了
});

```