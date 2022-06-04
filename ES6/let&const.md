
#### let/const

ES6提供了let/const命令声明变量。let声明的变量是有块级作用域的，cosnt用来声明常量。

var和let区别：
* var没有块级作用域，在全局环境声明会变成全局变量；
  ```js
  var a = [];
  for (var i = 0; i < 10; i++) {
    a[i] = function () {
      return i;
    };
  }
  console.log(window.a); // [ƒ, ƒ, ƒ, ƒ, ƒ, ƒ, ƒ, ƒ, ƒ, ƒ]
  console.log(a.map(item => item())); // [10, 10, 10, 10, 10, 10, 10, 10, 10, 10]
  ```

* let有块级作用域，在全局环境声明不会变成全局变量；
  ```js
  let a = [];
  for (let i = 0; i < 10; i++) {
    a[i] = function () {
      return i;
    };
  }
  console.log(window.a); // undefined
  console.log(a.map(item => item())); // [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
  ```

const声明的常量，值不能被改变。
```js
const test = 1;
test = 2; // Uncaught TypeError: Assignment to constant variable.
```

const变量在声明时就要初始化。
```js
const c; // Uncaught SyntaxError: Missing initializer in const declaration
```

const声明的对象的属性值可以改变。
```js
const d = { a: 1 };
d.a = 2;
console.log(d); // {a: 2}
```
