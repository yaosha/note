#### Symbol

Symbol是一个基本数据类型，Symbol()返回的值都是唯一的，唯一用途是作为对象属性的标识符。

参数可选，字符串类型。
```js
const a = Symbol('a');
const aa = Symbol('a');
a === aa; // false

// 不能作为构造函数使用
var sym = new Symbol(); // Uncaught TypeError: Symbol is not a constructor

// typeof可以识别Symbol类型
typeof Symbol() === 'symbol'
```

Symbols 在 for...in 迭代中不可枚举。另外，Object.getOwnPropertyNames() 不会返回 symbol 对象的属性，但是你能使用 Object.getOwnPropertySymbols() 得到它们。

当使用 JSON.stringify() 时，以 symbol 值作为键的属性会被完全忽略。

内置Symbol

迭代 symbols Symbol.iterator，被 for...of 使用。

自定义迭代器
```js
var myIterable = {}
myIterable[Symbol.iterator] = function* () {
    yield 1;
    yield 2;
    yield 3;
};
[...myIterable] // [1, 2, 3]
```