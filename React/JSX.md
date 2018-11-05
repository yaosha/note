## JSX

JSX编译后调用React.createElement(component, props, ...children)实现。

```javascript
<MyButton color="blue" shadowSize={2}>
  Click Me
</MyButton>
```

编译为

```javascript
React.createElement(
  MyButton,
  {color: 'blue', shadowSize: 2},
  'Click Me'
)
```

### JSX标签

JSX标签指定了React元素类型，大写开头是React组件，小写开头是html标签。

####React必须在作用域内

JSX编译后的代码使用React.createElement，所以React必须在作用域内。

#### JSX标签使用“.“语法

JSX中可以使用”.”。

```js
import React from 'react';

const MyComponents = {
  DatePicker: function DatePicker(props) {
    return <div>Imagine a {props.color} datepicker here.</div>;
  }
}

function BlueDatePicker() {
  return <MyComponents.DatePicker color="blue" />;
}
```

#### 自定义组件首字母必须大写

小写开头标签编译后是html标签名称的字符串。

大写开头标签编译后是一个组件对应的变量，所以组件必须在作用域内。

如果必须使用小写定义的组件，使用时必须赋值给大写变量，然后在JSX中使用。

#### 运行时选择使用的组件

JSX标签不能使用通用表达式，如果确实需要使用表达式确定使用的组件，将表达式赋值给首字母大写的变量。

错误示例：
```js
import React from 'react';
import { PhotoStory, VideoStory } from './stories';

const components = {
  photo: PhotoStory,
  video: VideoStory
};

function Story(props) {
  // Wrong! JSX type can't be an expression.
  return <components[props.storyType] story={props.story} />;
}
```

正确示例：

```js
import React from 'react';
import { PhotoStory, VideoStory } from './stories';

const components = {
  photo: PhotoStory,
  video: VideoStory
};

function Story(props) {
  // Correct! JSX type can be a capitalized variable.
  const SpecificStory = components[props.storyType];
  return <SpecificStory story={props.story} />;
}
```

### JSX的props

#### js表达式

使用{}包起来的表达式：

```js
<MyComponent foo={1 + 2 + 3 + 4} />
```

#### 字符串

字符串是转义的，下面示例都是等价的。

```js
<MyComponent message="hello world" />

<MyComponent message={'hello world'} />

<MyComponent message="&lt;3" />

<MyComponent message={'<3'} />
```

#### Props默认true

```js
<MyTextBox autocomplete /> //不建议使用这种方式

<MyTextBox autocomplete={true} />
```

#### 展开语法

```js
function App1() {
  return <Greeting firstName="Ben" lastName="Hector" />;
}

function App2() {
  const props = {firstName: 'Ben', lastName: 'Hector'};
  return <Greeting {...props} />;
}
```

谨慎使用，很容易传入组件不需要的props。

### Children

JSX标签开闭之间的内容自动视为this.props.children。

#### 字符串

```js
<MyComponent>Hello world!</MyComponent>
```

自动删除前后空格和空行，内容只加你的空行会转换为空格。

示例：

```html
<div>Hello World</div>

<div>
  Hello World
</div>

<div>
  Hello
  World
</div>

<div>

  Hello World
</div>
```

#### JSX

```html
<div>
  Here is a list:
  <ul>
    <li>Item 1</li>
    <li>Item 2</li>
  </ul>
</div>
```

React 16中可以使用数组

```js
render() {
  // No need to wrap list items in an extra element!
  return [
    // Don't forget the keys :)
    <li key="A">First item</li>,
    <li key="B">Second item</li>,
    <li key="C">Third item</li>,
  ];
}
```

#### js表达式

```js
function Item(props) {
  return <li>{props.message}</li>;
}

function TodoList() {
  const todos = ['finish doc', 'submit pr', 'nag dan to review'];
  return (
    <ul>
      {todos.map((message) => <Item key={message} message={message} />)}
    </ul>
  );
}
```

#### 函数

函数可以作为children，但是必须在render中转换为React可识别的内容。

```js
// Calls the children callback numTimes to produce a repeated component
function Repeat(props) {
  let items = [];
  for (let i = 0; i < props.numTimes; i++) {
    items.push(props.children(i));
  }
  return <div>{items}</div>;
}

function ListOfTenThings() {
  return (
    <Repeat numTimes={10}>
      {(index) => <div key={index}>This is item {index} in the list</div>}
    </Repeat>
  );
}
```

#### Boolean、null、undefined

Boolean、null、undefined会被忽略。

```html
//返回同样内容

<div />

<div></div>

<div>{false}</div>

<div>{null}</div>

<div>{undefined}</div>

<div>{true}</div>
```

使用这个特性可以实现条件判断渲染

```html
<div>
  {showHeader && <Header />}
  <Content />
</div>
```

如果需要false, true, null, 或 undefined出现在输出，转换为字符串。

```html
<div>
  My JavaScript variable is {String(myVariable)}.
</div>
```
