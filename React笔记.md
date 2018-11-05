# React基础

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

## 协调算法

随着props或state的变化，React在不同的时间点返回不同的React元素。React内部实现了协调算法比较每次变化，高效的更新组件。

为了实现更高效的协调算法，React定义了2个假设：
* 2个不同的元素产生不同的树
* 开发人员可以使用key提示React哪些组件在更新过程中是稳定的

### 比较不同元素

当元素类型不同时，React卸载旧元素，加载新元素。旧元素会执行componentWillUnmount()，新元素会执行componentWillMount() 和 componentDidMount()。

### 比较相同元素

#### 比较相同DOM节点

当DOM节点相同时，React比较属性、维持相同的底层DOM节点，只更新变化的属性。

```html
<div className="before" title="stuff" />

<div className="after" title="stuff" />

<div style={{color: 'red', fontWeight: 'bold'}} />

<div style={{color: 'green', fontWeight: 'bold'}} />
```

#### 比较相同的组件

相同组件更新时，组件实例是一样的，state的数据在内部维持。触发componentWillReceiveProps() 和componentWillUpdate()。
* 重复子组件：使用key标识子组件，key必须在循环范围内唯一、不变；不建议使用索引，索引可以作为最后的选择且前提是子组件顺序不变。

## Ref

React数据流中props从父组件传到子组件，但是有些场景需要在数据流外修改子组件。子组件可能是组件元素或DOM节点。React提供ref处理这种场景。

### 使用场景
* 管理focus、文本选中、或多媒体播放
* 触发必要的动画
* 集成其它三方DOM库

避免将refs用于可以声明性地完成的任何操作。

不要滥用refs

### 创建ref

React 16.3提供了新的API：React.createRef() 生成ref。

```js
class MyComponent extends React.Component {
  constructor(props) {
    super(props);
    this.myRef = React.createRef();
  }
  render() {
    return <div ref={this.myRef} />;
  }
}
```

### 访问refs

当ref在render中传递给元素，Ref的current属性指向节点。

```js
const node = this.myRef.current;
```

* 当ref用于HTML元素，current指向DOM元素
* 当ref用于自定义类组件，current指向组件实例
* 不能将ref指向函数式组件，因为不能实例化，但是可以在函数式组件内部使用ref指向DOM元素或自定义组件

示例1：ref指向DOM元素，当组件加载时，React将DOM元素赋给current，卸载时current置为null，ref的更新发生在componentDidMount 或 componentDidUpdate之前。

```js
class CustomTextInput extends React.Component {
  constructor(props) {
    super(props);
    // create a ref to store the textInput DOM element
    this.textInput = React.createRef();
    this.focusTextInput = this.focusTextInput.bind(this);
  }

  focusTextInput() {
    // Explicitly focus the text input using the raw DOM API
    // Note: we're accessing "current" to get the DOM node
    this.textInput.current.focus();
  }

  render() {
    // tell React that we want to associate the <input> ref
    // with the `textInput` that we created in the constructor
    return (
      <div>
        <input
          type="text"
          ref={this.textInput} />
        <input
          type="button"
          value="Focus the text input"
          onClick={this.focusTextInput}
        />
      </div>
    );
  }
}
```

示例2：ref指向自定义类组件

```js
class AutoFocusTextInput extends React.Component {
  constructor(props) {
    super(props);
    this.textInput = React.createRef();
  }

  componentDidMount() {
    this.textInput.current.focusTextInput();
  }

  render() {
    return (
      <CustomTextInput ref={this.textInput} />
    );
  }
}
```

示例3：函数式组件内部使用ref

```js
function CustomTextInput(props) {
  // textInput must be declared here so the ref can refer to it
  let textInput = React.createRef();

  function handleClick() {
    textInput.current.focus();
  }

  return (
    <div>
      <input
        type="text"
        ref={textInput} />
      <input
        type="button"
        value="Focus the text input"
        onClick={handleClick}
      />
    </div>
  );
}
```

### 暴露refs给父组件

在父组件访问子组件的DOM节点的ref，这种使用场景很稀少，不提倡这样，因为会打破组件的封装。

上面示例2不是好的解决办法，因为只得到组件的实例引用而不是DOM节点，而且函数式组件不支持ref。

如果React版本在16.3以上，使用React.forwardRef处理这种场景。

如果React版本在16.2以下，使用通过props传递ref的方式。

示例：通过props传递ref

```js
function CustomTextInput(props) {
  return (
    <div>
      <input ref={props.inputRef} />
    </div>
  );
}

function Parent(props) {
  return (
    <div>
      My input: <CustomTextInput inputRef={props.inputRef} />
    </div>
  );
}

class Grandparent extends React.Component {
  constructor(props) {
    super(props);
    this.inputElement = React.createRef();
  }
  render() {
    return (
      <Parent inputRef={this.inputElement} />
    );
  }
}
```

### 回调refs

另一种设置ref的方式。

示例：通过回调函数设置ref
```js
class CustomTextInput extends React.Component {
  constructor(props) {
    super(props);

    this.textInput = null;

    this.setTextInputRef = element => {
      this.textInput = element;
    };

    this.focusTextInput = () => {
      // Focus the text input using the raw DOM API
      if (this.textInput) this.textInput.focus();
    };
  }

  componentDidMount() {
    // autofocus the input on mount
    this.focusTextInput();
  }

  render() {
    // Use the `ref` callback to store a reference to the text input DOM
    // element in an instance field (for example, this.textInput).
    return (
      <div>
        <input
          type="text"
          ref={this.setTextInputRef}
        />
        <input
          type="button"
          value="Focus the text input"
          onClick={this.focusTextInput}
        />
      </div>
    );
  }
}
```

示例：通过回调函数传递ref
```js
function CustomTextInput(props) {
  return (
    <div>
      <input ref={props.inputRef} />
    </div>
  );
}

class Parent extends React.Component {
  render() {
    return (
      <CustomTextInput
        inputRef={el => this.inputElement = el}
      />
    );
  }
}
```

#### 注意事项

如果使用行内函数作为回调ref，在更新中会调用2次，第一次赋值null，第二次赋值DOM元素。这是因为每次render都会创建新的函数实例，所以React要清除旧ref设置新ref。

可以通过将ref回调定义为类的绑定方法来避免这种情况，但请注意，在大多数情况下它应该无关紧要。

### 旧版API：字符串refs

React之前版本可以给ref赋值字符串，使用this.refs.xxx访问。这种方式存在问题，禁止使用。

## 类型检查

使用`Flow`或`TypeScript`可以实现类型检查，如果不使用它们，React内置类型检查功能，通过`propTypes`属性检查props类型。需要导入`prop-types`。

### PropTypes

```javacript
import PropTypes from 'prop-types';

MyComponent.propTypes = {
  // 基本js类型
  optionalArray: PropTypes.array,
  optionalBool: PropTypes.bool,
  optionalFunc: PropTypes.func,
  optionalNumber: PropTypes.number,
  optionalObject: PropTypes.object,
  optionalString: PropTypes.string,
  optionalSymbol: PropTypes.symbol,

  // 可被渲染任意类型: numbers, strings, elements or an array
  // (or fragment) containing these types.
  optionalNode: PropTypes.node,

  // React元素
  optionalElement: PropTypes.element,

  // 定义某个类的实例，使用instanceof判断
  optionalMessage: PropTypes.instanceOf(Message),

  // 枚举
  optionalEnum: PropTypes.oneOf(['News', 'Photos']),

  // 类型种类范围
  optionalUnion: PropTypes.oneOfType([
    PropTypes.string,
    PropTypes.number,
    PropTypes.instanceOf(Message)
  ]),

  // 某个类型的数组
  optionalArrayOf: PropTypes.arrayOf(PropTypes.number),

  // 属性值为特定类型的对象
  optionalObjectOf: PropTypes.objectOf(PropTypes.number),

  // 指定key及value类型的对象
  optionalObjectWithShape: PropTypes.shape({
    color: PropTypes.string,
    fontSize: PropTypes.number
  }),

  // 使用`isRequired`指定props是必须的
  requiredFunc: PropTypes.func.isRequired,

  // 任意类型
  requiredAny: PropTypes.any.isRequired,

  // 自定义类型验证，验证不通过时返回Error对象
  customProp: function(props, propName, componentName) {
    if (!/matchme/.test(props[propName])) {
      return new Error(
        'Invalid prop `' + propName + '` supplied to' +
        ' `' + componentName + '`. Validation failed.'
      );
    }
  },

  // 在 `arrayOf` 和 `objectOf`也可以使用自定义验证
  // 验证不通过时返回Error对象
  // 验证数组或对象的每个可以
  // 第一个参数为对象或数组自身
  // 第二个参数为当前项的key
  customArrayProp: PropTypes.arrayOf(function(propValue, key, componentName, location, propFullName) {
    if (!/matchme/.test(propValue[key])) {
      return new Error(
        'Invalid prop `' + propFullName + '` supplied to' +
        ' `' + componentName + '`. Validation failed.'
      );
    }
  })
};
```

### 定义props默认值

使用`defaultProps`定义props的默认值。

```javascript
class Greeting extends React.Component {
  static defaultProps = {
    name: 'stranger'
  }

  render() {
    return (
      <div>Hello, {this.props.name}</div>
    )
  }
}
```

## props和state

### props是只读的

props作为组件的输入，对于相同的输入必须呈现相同的内容，所以组件内不能修改props。

### state

#### 禁止直接修改state

必须使用`setState`方法修改state。constructor是唯一可以给state直接赋值的地方。

```javascript
constructor(props) {
  super(props);
  this.state = {
    comment: ''
  };
}

this.setState({comment: 'Hello'});
```

#### state的更新是异步的

React为了性能会将多个setState按批次执行。所以不能依赖它们的值计算下次的state。

错误的使用方式
```javascript
this.setState({
  counter: this.state.counter + this.props.increment,
});
```
正确的使用方式
```javascript
this.setState((state, props) => ({
  counter: state.counter + props.increment
}));
```
#### state更新会合并

当调用`setState()`，Reac会将对象合并到当前state。

state包含多个独立变量
```javascript
constructor(props) {
  super(props);
  this.state = {
    posts: [],
    comments: []
  };
}
```

然后分别调用`setState()`更新
```javascript
  componentDidMount() {
    fetchPosts().then(response => {
      this.setState({
        posts: response.posts
      });
    });

    fetchComments().then(response => {
      this.setState({
        comments: response.comments
      });
    });
  }
```

state合并是浅层的，`this.setState({posts})`只会替换`this.state.posts`，不会影响`this.state.comments`。

### 数据向下流动

组件被使用时，不关心是否包换state，也不关心是类组件还是函数组件。

state封装在组件内部，除了拥有它的组件不会被其它任何组件访问。

父组件可以将其state设置为子组件的props。子组件不关心这个props是否是父组件的state。

这通常被称为`自上而下`或`单向`数据流。任何state始终由某个组件拥有，任何从这个state派生的数据或UI只能影响层级处于下方的组件。

## 事件处理

React的事件绑定和HTML中有点区别：
* React事件使用驼峰命名
* 在JSX中传递函数作为事件处理器
* 不能通过返回false阻止事件传播，必须调用preventDefault 

HTML
```html
<button onclick="activateLasers()">
  Activate Lasers
</button>
```

JXX
```html
<button onClick={activateLasers}>
  Activate Lasers
</button>
```

### 绑定事件处理函数

```js
class Toggle extends React.Component {
  constructor(props) {
    super(props);
    this.state = {isToggleOn: true};

    // This binding is necessary to make `this` work in the callback
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    this.setState(state => ({
      isToggleOn: !state.isToggleOn
    }));
  }

  render() {
    return (
      <button onClick={this.handleClick}>
        {this.state.isToggleOn ? 'ON' : 'OFF'}
      </button>
    );
  }
}

ReactDOM.render(
  <Toggle />,
  document.getElementById('root')
);
```

将事件处理函数作为类的方法。类方法默认不会自动绑定到`this`，当像这样`onClick={this.handleClick}`使用时，必须使用`bind`绑定方法到`this`。

如果不想使用`bingd`绑定函数，可以定义箭头函数。
```js
class LoggingButton extends React.Component {
  // This syntax ensures `this` is bound within handleClick.
  // Warning: this is *experimental* syntax.
  handleClick = () => {
    console.log('this is:', this);
  }

  render() {
    return (
      <button onClick={this.handleClick}>
        Click me
      </button>
    );
  }
}
```

### 传递额外参数到事件处理函数

如果在循环中，通常需要传递额外参数到事件处理函数。
```html
<button onClick={(e) => this.deleteRow(id, e)}>Delete Row</button>
<button onClick={this.deleteRow.bind(this, id)}>Delete Row</button>
```

两种方式是等价的，但是第一种在不建议在父组件中使用，因为会导致子组件重新渲染。

## 条件渲染

### if else

根据条件呈现不同内容，JSX短语可以赋值给变量。

```js
class LoginControl extends React.Component {
  constructor(props) {
    super(props);
    this.handleLoginClick = this.handleLoginClick.bind(this);
    this.handleLogoutClick = this.handleLogoutClick.bind(this);
    this.state = {isLoggedIn: false};
  }

  handleLoginClick() {
    this.setState({isLoggedIn: true});
  }

  handleLogoutClick() {
    this.setState({isLoggedIn: false});
  }

  render() {
    const isLoggedIn = this.state.isLoggedIn;
    let button;

    if (isLoggedIn) {
      button = <LogoutButton onClick={this.handleLogoutClick} />;
    } else {
      button = <LoginButton onClick={this.handleLoginClick} />;
    }

    return (
      <div>
        <Greeting isLoggedIn={isLoggedIn} />
        {button}
      </div>
    );
  }
}

ReactDOM.render(
  <LoginControl />,
  document.getElementById('root')
);
```

### 行内&&操作

```js
function Mailbox(props) {
  const unreadMessages = props.unreadMessages;
  return (
    <div>
      <h1>Hello!</h1>
      {unreadMessages.length > 0 &&
        <h2>
          You have {unreadMessages.length} unread messages.
        </h2>
      }
    </div>
  );
}

const messages = ['React', 'Re: React', 'Re:Re: React'];
ReactDOM.render(
  <Mailbox unreadMessages={messages} />,
  document.getElementById('root')
);
```

### 三元操作符

```js
render() {
  const isLoggedIn = this.state.isLoggedIn;
  return (
    <div>
      The user is <b>{isLoggedIn ? 'currently' : 'not'}</b> logged in.
    </div>
  );
}
```

### 阻止组件渲染

render中返回null隐藏组件内容。

```js
function WarningBanner(props) {
  if (!props.warn) {
    return null;
  }

  return (
    <div className="warning">
      Warning!
    </div>
  );
}

class Page extends React.Component {
  constructor(props) {
    super(props);
    this.state = {showWarning: true};
    this.handleToggleClick = this.handleToggleClick.bind(this);
  }

  handleToggleClick() {
    this.setState(state => ({
      showWarning: !state.showWarning
    }));
  }

  render() {
    return (
      <div>
        <WarningBanner warn={this.state.showWarning} />
        <button onClick={this.handleToggleClick}>
          {this.state.showWarning ? 'Hide' : 'Show'}
        </button>
      </div>
    );
  }
}

ReactDOM.render(
  <Page />,
  document.getElementById('root')
);
```

## 列表和key

### 循环渲染多个组件

```js
function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    <li>{number}</li>
  );
  return (
    <ul>{listItems}</ul>
  );
}

const numbers = [1, 2, 3, 4, 5];
ReactDOM.render(
  <NumberList numbers={numbers} />,
  document.getElementById('root')
);
```

以上代码在控制台会告警缺少key。

更正后的代码：

```js
function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    <li key={number.toString()}>
      {number}
    </li>
  );
  return (
    <ul>{listItems}</ul>
  );
}

const numbers = [1, 2, 3, 4, 5];
ReactDOM.render(
  <NumberList numbers={numbers} />,
  document.getElementById('root')
);
```

### keys

key帮助React识别循环中哪些元素修改、增加或删除。key必须使用唯一性的字符串（非全局，循环范围内）。常用方式使用id。

```js
const todoItems = todos.map((todo) =>
  <li key={todo.id}>
    {todo.text}
  </li>
);
```

如果没有其它选择，索引是最后的手段，但是不建议使用。

# React 16 变更

## Lazy-load

避免打包代码体积太大，使用webpack分割代码打包成多个文件，在运行时动态加载。

### import()

使用import()语法，webpack遇到这种语法会自动分割代码。

```js
import("./math").then(math => {
  console.log(math.add(16, 26));
});
```

### React.lazy

React.lazy方法允许将动态导入呈现为常规组件。

```js
const OtherComponent = React.lazy(() => import('./OtherComponent'));

function MyComponent() {
  return (
    <div>
      <OtherComponent />
    </div>
  );
}
```

React.lazy参数内必须调用动态import()，返回Promise，Promise中解析一个模块，这个模块带有default导出的React组件。

#### Suspense

在React.lazy导入的组件还未加载时，使用Suspense呈现替代内容，如加载框。

```js
const OtherComponent = React.lazy(() => import('./OtherComponent'));

function MyComponent() {
  return (
    <div>
      <Suspense fallback={<div>Loading...</div>}>
        <OtherComponent />
      </Suspense>
    </div>
  );
}
```

fallback接受React元素。Suspense可以在懒加载组件之上的任意位置，也可以包含多个懒加载组件。

#### 错误边界

当模块加载失败时，使用错误边界处理错误。错误边界位于懒加载组件之上任意位置。

```js
import MyErrorBoundary from '/MyErrorBoundary';
const OtherComponent = React.lazy(() => import('./OtherComponent'));
const AnotherComponent = React.lazy(() => import('./AnotherComponent'));

const MyComponent = () => (
  <div>
    <MyErrorBoundary>
      <Suspense fallback={<div>Loading...</div>}>
        <section>
          <OtherComponent />
          <AnotherComponent />
        </section>
      </Suspense>
    </MyErrorBoundary>
  </div>
);
```

### 基于路由的代码分割

```js
import { BrowserRouter as Router, Route, Switch } from 'react-router-dom';
import React, { Suspense, lazy } from 'react';

const Home = lazy(() => import('./routes/Home'));
const About = lazy(() => import('./routes/About'));

const App = () => (
  <Router>
    <Suspense fallback={<div>Loading...</div>}>
      <Switch>
        <Route exact path="/" component={Home}/>
        <Route path="/about" component={About}/>
      </Switch>
    </Suspense>
  </Router>
);
```

### 注意事项

React.lazy只支持default导出。如果需要使用命名导出的组件，创建一个模块使用default导出。

```js
// ManyComponents.js
export const MyComponent = /* ... */;
export const MyUnusedComponent = /* ... */;

// MyComponent.js
export { MyComponent as default } from "./ManyComponents.js";

// MyApp.js
import React, { lazy } from 'react';
const MyComponent = lazy(() => import("./MyComponent.js"));
```

## Context

Context可以视作全局变量，用于跨组件传递数据。
之前版本遗留的context API在v16中支持，在下个大版本移除。

### API

#### React.createContext

创建context对象，React组件通过匹配最近层级的Provider读取这个context，defaultValue用于在组件找不到匹配的Provider的情况。

```js
const myContext = React.createContext(defaultValue);
```

#### Context.Provider

value作为传递给Provider的子组件的值。当provider的value改变时，所有子组件都会重新渲染。不受shouldComponentUpdate的影响。使用Object.is比较value的值。

```html
<MyContext.Provider value={/* some value */}>
```

#### Class.contextType

将context对象赋值给组件类的contextType属性，组件内部可以使用this.context取得最近层级Context对象的当前值。

示例1：单个context
```js
class MyClass extends React.Component {
  componentDidMount() {
    let value = this.context;
    /* perform a side-effect at mount using the value of MyContext */
  }
  componentDidUpdate() {
    let value = this.context;
    /* ... */
  }
  componentWillUnmount() {
    let value = this.context;
    /* ... */
  }
  render() {
    let value = this.context;
    /* render something based on the value of MyContext */
  }
}
MyClass.contextType = MyContext;
```

示例2：单个context，static语法
```js
class MyClass extends React.Component {
  static contextType = MyContext;
  render() {
    let value = this.context;
    /* render something based on the value */
  }
}
```

示例3：多个context
```js
// Theme context, default to light theme
const ThemeContext = React.createContext('light');

// Signed-in user context
const UserContext = React.createContext({
  name: 'Guest',
});

class App extends React.Component {
  render() {
    const {signedInUser, theme} = this.props;

    // App component that provides initial context values
    // highlight-range{2-3,5-6}
    return (
      <ThemeContext.Provider value={theme}>
        <UserContext.Provider value={signedInUser}>
          <Layout />
        </UserContext.Provider>
      </ThemeContext.Provider>
    );
  }
}

function Layout() {
  return (
    <div>
      <Sidebar />
      <Content />
    </div>
  );
}

// A component may consume multiple contexts
function Content() {
  // highlight-range{2-10}
  return (
    <ThemeContext.Consumer>
      {theme => (
        <UserContext.Consumer>
          {user => (
            <ProfilePage user={user} theme={theme} />
          )}
        </UserContext.Consumer>
      )}
    </ThemeContext.Consumer>
  );
}
```

如果有多个经常一起使用的context，可以合并成一个。

#### Context.Consumer

```html
<MyContext.Consumer>
  {value => /* render something based on the context value */}
</MyContext.Consumer>
```

订阅context变化的React组件，可以在函数式组件内部订阅context。

### 注意事项

使用对象作为provider的value时，必须使用变量。不可以直接赋值{…}，因为每次都是新对象，会触发render。

## 错误边界

React16使用错误边界处理组件错误，避免整个组件树崩溃。

错误边界捕获其子组件任意地方（render、生命周期、构造函数）的javascript错误、记录日志、显示反馈UI。但是不处理如下错误：
* 事件处理
* 异步代码（例如setTimeout、requestAnimationFrame回调）
* 服务端渲染
* 错误边界自身抛出的错误

定义生命周期方法static getDerivedStateFromError()、componentDidCatch()两者之一或都定义就是一个错误边界。

### 定义

#### static getDerivedStateFromError()

发生错误时的呈现的界面。

#### componentDidCatch()

记录错误信息

#### 示例

```js
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    // Update state so the next render will show the fallback UI.
    return { hasError: true };
  }

  componentDidCatch(error, info) {
    // You can also log the error to an error reporting service
    logErrorToMyService(error, info);
  }

  render() {
    if (this.state.hasError) {
      // You can render any custom fallback UI
      return <h1>Something went wrong.</h1>;
    }

    return this.props.children; 
  }
}
```

### 用法

#### 使用位置

错误边界的粒度取决于使用场景。

应用最外层：将顶级路由组件包装为向用户显示错误消息，就像服务器端框架处理崩溃一样。

组件粒度：将各个小部件包装在错误边界中，以防止它们崩溃应用程序的其余部分。

#### 错误未捕获的后果

未被错误边界捕获的错误会导致卸载整个React组件树。

### API的改变

React 15有限支持错误处理的方法unstable_handleError在16中已不再支持，需要更改为componentDidCatch。

## Fragment

React.Fragment是一个内置组件，用于返回多个元素但是不增加额外的DOM。

```js
render() {
  return (
    <React.Fragment>
      <ChildA />
      <ChildB />
      <ChildC />
    </React.Fragment>
  );
}
```

界面实际上展示的：
```html
<ChildA />
<ChildB />
<ChildC />
```

唯一可使用的属性是key，用于在列表循环中使用。
```js
function Glossary(props) {
  return (
    <dl>
      {props.items.map(item => (
        // Without the `key`, React will fire a key warning
        <React.Fragment key={item.id}>
          <dt>{item.term}</dt>
          <dd>{item.description}</dd>
        </React.Fragment>
      ))}
    </dl>
  );
}
```

## ReactDOM.createPortal

ReactDOM.createPortal可以将children呈现到父组件层级外的DOM节点。

* child：可被渲染的React元素
* container：DOM元素

```js
ReactDOM.createPortal(child, container)
```

### 使用场景

通常来说，render返回的DOM会加到最近的父级节点，但是有时候需要将节点加到DOM树中另外的节点。

示例：创建div呈现children
```js
render() {
  return (
    <div>
      {this.props.children}
    </div>
  );
}
```

示例：不创建div，而是使用domNode呈现children，domNode可以是任意位置的有效DOM节点
```js
render() {
  // React does *not* create a new div. It renders the children into `domNode`.
  // `domNode` is any valid DOM node, regardless of its location in the DOM.
  return ReactDOM.createPortal(
    this.props.children,
    domNode
  );
}
```

### 事件冒泡

不管portal加到哪个节点，它还是处于React树中，所以和其它正常React元素一样。

Portal中的事件还是会冒泡到React树中，不管其是不是Portal的祖先节点。

```html
<html>
  <body>
    <div id="app-root"></div>
    <div id="modal-root"></div>
  </body>
</html>
```

React层级：Parent>Modal>Child

DOM层级：app-root>Parent;modal-root>Child

尽管Child不是app-root的子节点，但是事件会冒到Parent中。

```js
// app-root和modal-root是兄弟节点
const appRoot = document.getElementById('app-root');
const modalRoot = document.getElementById('modal-root');

class Modal extends React.Component {
  constructor(props) {
    super(props);
    this.el = document.createElement('div');
  }

  componentDidMount() {
    // 在Modal的children加载后portal插入到DOM树中，
    // 意味着children加载到分离的DOM节点。
    // 如果子组件必须在加载后必须立即附加到DOM树，
    // 例如衡量一个DOM节点或在后代元素使用'autoFocus'，
    // 则在Modal添加state，并仅在DOM树插入Modal后渲染children。
    modalRoot.appendChild(this.el);
  }

  componentWillUnmount() {
    modalRoot.removeChild(this.el);
  }

  render() {
    return ReactDOM.createPortal(
      this.props.children,
      this.el,
    );
  }
}

class Parent extends React.Component {
  constructor(props) {
    super(props);
    this.state = {clicks: 0};
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    // 当Child的按钮点击触发，更新Parent的state，尽管按钮不是其直接后代元素。
    this.setState(state => ({
      clicks: state.clicks + 1
    }));
  }

  render() {
    return (
      <div onClick={this.handleClick}>
        <p>Number of clicks: {this.state.clicks}</p>
        <p>
          Open up the browser DevTools
          to observe that the button
          is not a child of the div
          with the onClick handler.
        </p>
        <Modal>
          <Child />
        </Modal>
      </div>
    );
  }
}

function Child() {
  // 按钮点击事件会冒泡到父级，因为这里没有定义onClick属性。
  return (
    <div className="modal">
      <button>Click</button>
    </div>
  );
}

ReactDOM.render(<Parent />, appRoot);
```

## StrictMode;

React.StrictMode和Fragement一样，不呈现任何UI，它会激活后台组件中额外检查和告警，检查应用中潜在的问题。

> 注意
>
> 严格模式检查只会在开发模式中运行，不影响生产构建。

使用示例

```javascript
import React from 'react';

function ExampleApplication() {
  return (
    <div>
      <Header />
      <React.StrictMode>
        <div>
          <ComponentOne />
          <ComponentTwo />
        </div>
      </React.StrictMode>
      <Footer />
    </div>
  );
}
```

检查范围

* 检查是否存在遗留的React生命周期函数
* 检查是否存在遗留的字符串ref
* 检查是否存在遗留的findDOMNode方法
* 检查是否存在遗留的context使用方式
* 检查意外的副作用

### 检查意外的副作用

React运行分2个阶段：

* 渲染：确定需要改变的内容。在此阶段，调用render，然后将结果和之前进行比较。
* 提交：React应用任何改变。在次阶段，React还会调用componentDidMount和componentDidUpdate之类的生命周期。

提交阶段通常非常快，但渲染速度可能很慢。 出于这个原因，即将推出的异步模式（默认情况下尚未启用）会将渲染工作分解为多个部分，以避免阻塞浏览器。 这意味着React可以在提交之前多次调用渲染阶段生命周期，或者它可以在不提交的情况下调用它们（因为错误或更高优先级的中断）。

渲染阶段生命周期包括以下类组件方法：

* constructor
* componentWillMount
* componentWillReceiveProps
* componentWillUpdate
* getDerivedStateFromProps
* shouldComponentUpdate
* render
* setState updater functions (the first argument)

因为以上方法可能调用多次，所以确保它们多次调用没有副作用。StrictMode通过双重调用以下方法检查副作用：

* constructor
* render
* setState updater functions (the first argument)
* getDerivedStateFromProps

# React技巧

## HOC

高阶组件（higher-order component）是React复用组件逻辑的高级技巧。高阶组件是一个使用组件作为参数返回新组件的函数。

### 分离关注点

如果多个组件拥有相同的处理逻辑，可以提取相同的逻辑到HOC实现。

HOC不修改源组件，也不继承源组件。HOC将源组件包装到容器组件。HOC是没有副作用的纯函数。

HOC不关心props如何使用，源组件不关心props来源。

### 不要修改源组件，使用组合

在HOC内部不要修改源组件的prototype，这会导致源组件行为改变，而且如果使用另外的HOC包装前一个HOC，代码会被覆盖。

### 使用惯例

#### 透传HOC不关注的props到源组件

HOC不应该修改源组件的对外接口，透传与特定关注点无关的props。
```js
render() {
  // Filter out extra props that are specific to this HOC and shouldn't be
  // passed through
  const { extraProp, ...passThroughProps } = this.props;

  // Inject props into the wrapped component. These are usually state values or
  // instance methods.
  const injectedProp = someStateOrInstanceMethod;

  // Pass props to wrapped component
  return (
    <WrappedComponent
      injectedProp={injectedProp}
      {...passThroughProps}
    />
  );
}
```

#### 最大化可组合性

HOC最常用签名【Component => Component】，如redux：
```js
// React Redux's `connect`
const ConnectedComment = connect(commentSelector, commentActions)(CommentList);
connect(commentSelector, commentActions)的返回值是一个HOC。
3.1.3.3 包装displayName以方便调试
function withSubscription(WrappedComponent) {
  class WithSubscription extends React.Component {/* ... */}
  WithSubscription.displayName = `WithSubscription(${getDisplayName(WrappedComponent)})`;
  return WithSubscription;
}

function getDisplayName(WrappedComponent) {
  return WrappedComponent.displayName || WrappedComponent.name || 'Component';
}
```

### 注意事项

#### 不要在render内部使用HOC

React的一致性算法会比较render的返回值，如果在render内部使用HOC，每次render的返回值都是不一样的会导致每次都mount和unmount，除了性能问题还会导致state丢失。

错误示例：
```js
render() {
  // A new version of EnhancedComponent is created on every render
  // EnhancedComponent1 !== EnhancedComponent2
  const EnhancedComponent = enhance(MyComponent);
  // That causes the entire subtree to unmount/remount each time!
  return <EnhancedComponent />;
}
```

#### 复制静态方法

HOC返回的组件不包含源组件的静态方法，需要在HOC方法内将静态方法复制到返回的组件。

示例1：需要知道静态方法名
```js
function enhance(WrappedComponent) {
  class Enhance extends React.Component {/*...*/}
  // Must know exactly which method(s) to copy :(
  Enhance.staticMethod = WrappedComponent.staticMethod;
  return Enhance;
}
```

示例2：使用hoist-non-react-statics，自动复制非React的静态方法
```js
import hoistNonReactStatic from 'hoist-non-react-statics';
function enhance(WrappedComponent) {
  class Enhance extends React.Component {/*...*/}
  hoistNonReactStatic(Enhance, WrappedComponent);
  return Enhance;
}
```

示例3：源组件导出静态方法
```js
// Instead of...
MyComponent.someFunction = someFunction;
export default MyComponent;

// ...export the method separately...
export { someFunction };

// ...and in the consuming module, import both
import MyComponent, { someFunction } from './MyComponent.js';
```

#### Ref不会传递到源组件

使用React.forwardRef转发ref。

## 集成其它库

### 集成DOM操作插件

React内部机制决定是否更新DOM，如果其它库操作DOM，React是不知道的引发不可知的结果。

但是结合其它库一起开发业是可以的，最容易的方式就是阻止React更新DOM。

#### 阻止React更新

Render中返回空DIV，React不会触发更新。componentDidMount中初始化插件，componentWillUnmount中销毁插件。

示例
```js
class SomePlugin extends React.Component {
  componentDidMount() {
    this.$el = $(this.el);
    this.$el.somePlugin();
  }

  componentWillUnmount() {
    this.$el.somePlugin('destroy');
  }

  render() {
    return <div ref={el => this.el = el} />;
  }
}
```

#### 如果触发更新

实现componentDidUpdate，处理更新的属性。

示例
```js
class Chosen extends React.Component {
  componentDidMount() {
    this.$el = $(this.el);
    this.$el.chosen();

    this.handleChange = this.handleChange.bind(this);
    this.$el.on('change', this.handleChange);
  }
  
  componentDidUpdate(prevProps) {
    if (prevProps.children !== this.props.children) {
      this.$el.trigger("chosen:updated");
    }
  }

  componentWillUnmount() {
    this.$el.off('change', this.handleChange);
    this.$el.chosen('destroy');
  }
  
  handleChange(e) {
    this.props.onChange(e.target.value);
  }

  render() {
    return (
      <div>
        <select className="Chosen-select" ref={el => this.el = el}>
          {this.props.children}
        </select>
      </div>
    );
  }
}
```

### 集成其它视图库

#### 使用React替换基于字符串的HTML

使用ReactDOM.render方法替换innerHTML。

以下代码
```js
$('#container').html('<button id="btn">Say Hello</button>');
$('#btn').click(function() {
  alert('Hello!');
});
```

可以被替换为
```js
function Button(props) {
  return <button onClick={props.onClick}>Say Hello</button>;
}

function HelloButton() {
  function handleClick() {
    alert('Hello!');
  }
  return <Button onClick={handleClick} />;
}

ReactDOM.render(
  <HelloButton />,
  document.getElementById('container')
);
```

#### 嵌入到Backbone视图

重写Backbone的render方法，在remove中调用unmountComponentAtNode卸载组件清除资源。

```js
function Paragraph(props) {
  return <p>{props.text}</p>;
}

const ParagraphView = Backbone.View.extend({
  render() {
    const text = this.model.get('text');
    ReactDOM.render(<Paragraph text={text} />, this.el);
    return this;
  },
  remove() {
    ReactDOM.unmountComponentAtNode(this.el);
    Backbone.View.prototype.remove.call(this);
  }
});
```

### 集成模型层

#### 在React中使用Backbone模型

监听模型改变事件，强制触发更新。
```js
class Item extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
  }

  handleChange() {
    this.forceUpdate();
  }

  componentDidMount() {
    this.props.model.on('change', this.handleChange);
  }

  componentWillUnmount() {
    this.props.model.off('change', this.handleChange);
  }

  render() {
    return <li>{this.props.model.get('text')}</li>;
  }
}

class List extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
  }

  handleChange() {
    this.forceUpdate();
  }

  componentDidMount() {
    this.props.collection.on('add', 'remove', this.handleChange);
  }

  componentWillUnmount() {
    this.props.collection.off('add', 'remove', this.handleChange);
  }

  render() {
    return (
      <ul>
        {this.props.collection.map(model => (
          <Item key={model.cid} model={model} />
        ))}
      </ul>
    );
  }
}
```

#### 从Backbone提取模型

使用HOC在生命周期函数中订阅模型的改变，复制数据到state。

```js
function connectToBackboneModel(WrappedComponent) {
  return class BackboneComponent extends React.Component {
    constructor(props) {
      super(props);
      this.state = Object.assign({}, props.model.attributes);
      this.handleChange = this.handleChange.bind(this);
    }

    componentDidMount() {
      this.props.model.on('change', this.handleChange);
    }

    componentWillReceiveProps(nextProps) {
      this.setState(Object.assign({}, nextProps.model.attributes));
      if (nextProps.model !== this.props.model) {
        this.props.model.off('change', this.handleChange);
        nextProps.model.on('change', this.handleChange);
      }
    }

    componentWillUnmount() {
      this.props.model.off('change', this.handleChange);
    }

    handleChange(model) {
      this.setState(model.changedAttributes());
    }

    render() {
      const propsExceptModel = Object.assign({}, this.props);
      delete propsExceptModel.model;
      return <WrappedComponent {...propsExceptModel} {...this.state} />;
    }
  }
}
```

## render props

render props是一个函数类型的props，名称为render（或随意命名），用于动态呈现组件内容。React Router使用这种方式。

### 分离关注点

组件是React内代码复用单元，但是如何分享组件state或props到另一个需要这些数据的组件也是一个问题。

使用render props动态渲染内容，达到复用父级组件的目的。

示例：复用鼠标移动代码

```js
class Cat extends React.Component {
  render() {
    const mouse = this.props.mouse;
    return (
      <img src="/cat.jpg" style={{ position: 'absolute', left: mouse.x, top: mouse.y }} />
    );
  }
}

class Mouse extends React.Component {
  constructor(props) {
    super(props);
    this.handleMouseMove = this.handleMouseMove.bind(this);
    this.state = { x: 0, y: 0 };
  }

  handleMouseMove(event) {
    this.setState({
      x: event.clientX,
      y: event.clientY
    });
  }

  render() {
    return (
      <div style={{ height: '100%' }} onMouseMove={this.handleMouseMove}>

        {/*
          Instead of providing a static representation of what <Mouse> renders,
          use the `render` prop to dynamically determine what to render.
        */}
        {this.props.render(this.state)}
      </div>
    );
  }
}

class MouseTracker extends React.Component {
  render() {
    return (
      <div>
        <h1>Move the mouse around!</h1>
        <Mouse render={mouse => (
          <Cat mouse={mouse} />
        )}/>
      </div>
    );
  }
}
```

使用render props常规组件可以实现大多数HOC。
```js
// If you really want a HOC for some reason, you can easily
// create one using a regular component with a render prop!
function withMouse(Component) {
  return class extends React.Component {
    render() {
      return (
        <Mouse render={mouse => (
          <Component {...this.props} mouse={mouse} />
        )}/>
      );
    }
  }
}
```

### Children props

也可以使用children实现render props。
```html
<Mouse children={mouse => (
  <p>The mouse position is {mouse.x}, {mouse.y}</p>
)}/>

<Mouse>
  {mouse => (
    <p>The mouse position is {mouse.x}, {mouse.y}</p>
  )}
</Mouse>
```

必须定义children类型为函数
```js
Mouse.propTypes = {
  children: PropTypes.func.isRequired
};
```

### 注意事项

#### Render props和React.PureComponent一起使用时要小心

如果在render中使用函数体，每次都是一个新的函数，PureComponent的浅层比较总是返回false。应该定义类方法替代函数体。

错误示例
```js
class Mouse extends React.PureComponent {
  // Same implementation as above...
}

class MouseTracker extends React.Component {
  render() {
    return (
      <div>
        <h1>Move the mouse around!</h1>

        {/*
          This is bad! The value of the `render` prop will
          be different on each render.
        */}
        <Mouse render={mouse => (
          <Cat mouse={mouse} />
        )}/>
      </div>
    );
  }
}
```

正确示例
```js
class MouseTracker extends React.Component {
  // Defined as an instance method, `this.renderTheCat` always
  // refers to *same* function when we use it in render
  renderTheCat(mouse) {
    return <Cat mouse={mouse} />;
  }

  render() {
    return (
      <div>
        <h1>Move the mouse around!</h1>
        <Mouse render={this.renderTheCat} />
      </div>
    );
  }
}
```

Render props尽量结合React.Component使用。

## 组件分解

### 将UI分解为组件层级结构

* 单一指责原则，一个组件只做一件事，如果它最终增长，应该分解成更小的子组件。
* 按照数据结构分解，每个组件展示一部分数据。


### 构建静态版本

* 不考虑交互，不使用state，通过props传递数据
* 简单页面从上到下实现组件，复杂页面从下到上实现组件
* 修改最外层数据，检查数据流动情况


### 识别UI中最小但完整的state

* 列出最小的易变数据集合
* 关键是不要重复；找出应用需要的绝对最少的state，然后按需计算其它数据

* 是否通过props从父组件传进来？如果是就不是state
* 随着时间过去是否保持不变？如果是就不是state
* 是否可以通过其它props或state计算出来？如果是就不是state


### 识别state属于哪个组件

* 标识根据state呈现内容的每个组件
* 查找公共所有者组件（在层次结构中所有需要state组件上方的单个组件）
* 公共所有者组件或层次结构更高的其它组件应该拥有这个state
* 如果找不到应该拥有state的组件，创建一个包含这个state的简单组件，并将其添加到层级高于公共所有者组件的某个位置


### 添加反向数据流

* 子组件需要改变父级组件state，子组件定义时间回调props
* 父组件传递回调函数给子组件，函数值调用setState修改state

## 性能优化

### 使用生产构建，性能测试时，确保使用压缩的生产版本测试。

* 为什么？因为开发模式有许多帮助警告，导致react大且慢。
* 如何检查是否处于生产模式？安装chrome插件react-developer-tools，生产模式时，插件背景为黑色，开发模式时背景为红色。
* 方式
  * 使用create-react-app构建项目，运行npm run build构建生成模式代码
  * 单文件构建，引用react.production.min.js和react-dom.production.min.js文件
  * webpack
    * 使用new webpack.DefinePlugin定义process.env.NODE_ENV为production
    * 使用new webpack.optimize.UglifyJsPlugin()


### 使用Chrome Performance选项卡分析组件，

* 作用：
  * 在开发模式，可视化组件的安装，更新和卸载方式
  * 可以帮助了解不相关的UI何时被错误更新，以及UI更新发生的深度和频率。
* 步骤
  * 暂时禁用所有chrome插件，特别是React DevTools
  * 确保在开发模式运行项目
  * 打开Chrome Performance选项卡，按下Record
  * 执行要分析的操作，不要记录超过20秒否则Chrome可能会挂起。
  * 停止记录
  * React事件将在User Timing标签下分组。


### 使用DevTools Profiler分析组件

* 使用React DevTools Profiler，react-dom 16.5+和react-native 0.57+在DEV模式下提供增强的性能分析功能

### 虚拟化长列表
* 场景：呈现长列表（成百上千行）
* 技巧：仅在任何给定时间呈现行的一小部分
* 优点：显著减少重新呈现组件所花费的时间以及创建的DOM节点数
* 示例： react-virtualized


### 避免执行协调算法
* 虚拟DOM：React构建并维护呈现UI的内部展示，包括从组建返回的React元素
  * 优点：避免创建DOM节点并在必要时访问现有节点
  * 更新：当组建props或state改变时，React通过比较新返回的和上次呈现的元素决定是否需要实际的DOM更新。不等时，更新DOM。
* 通过 React DevTools可以可视化虚拟DOM的重渲染
  * 在React选项卡选中"Highlight Updates"
  * 与页面交互时，任何已重新渲染的组件周围会出现彩色边框
* "浪费"渲染现象：数据实际未改变，组件还是重新渲染
  * 未感知到速度减慢：忽略这点
  * 感知到速度减慢：
    * 覆写 shouldComponentUpdate加速
    * 继承React.PureComponent，等价于在shouldComponentUpdate中实现当前和上次props和state的浅层比较


### shouldComponentUpdate
* shouldComponentUpdate返回true，虚拟DOM不同，需要协调
* shouldComponentUpdate返回false，不需要协调
* shouldComponentUpdate返回true，虚拟DOM相同，不需要协调


### 不变数据的力量
* 浅层比较在比较对象时会导致问题，例如数组元素长度发生变化，但是传入的引用不变导致比较相同，React不会触发更新
  * setState避免使用上次state的变量
    * 数组
      * 使用concat： state.words.concat(['marklar'])
      * 展开语法：[...state.words, 'marklar']
    * 对象
      * Object.assign：Object.assign({}, colormap, {right: 'blue'});
      * 展开语法：{...colormap, right: 'blue'}

### 使用不变数据结构

除了上面的方法，还可以使用Immutable.js

* Immutable.js：通过结构共享提供不变的、持久的集合
  * 不可变：一旦创建，不能再另一个时间点改变
  * 持久性：可以从先前的集合或诸如set创建新集合，新集合创建后，原始集合仍然有效。
  * 结构共享：使用与原始集合尽可能相同的结构创建新集合，从而将复制消耗降至最低，以提高性能。
* 好处：使跟踪变更更方便，改变总是在新对象，只需比较检查新对象引用是否改变。

```js
const SomeRecord = Immutable.Record({ foo: null });
const x = new SomeRecord({ foo: 'bar' });
const y = x.set('foo', 'baz');
const z = x.set('foo', 'bar');
x === y; // false
x === z; // true
```

## 提升state

多个组件需要反映相同的state时，建议将共享state提升到最近的共同祖先组件。

对于React应用中发生变化的任何数据，应当只有唯一一个“真实来源”。

在开发过程中，某个组件需要通过state维持内部变化数据。之后有另外的组件也需要这个state，通过将它提升到最近的共同祖先达到共享的目的。应该依赖自上而下的数据流，而不是尝试在不同组件之间同步状态。

如果某些数据可以从props或state衍生，这些数据不应该放在state内。

## 组合

props可以传入React元素，由此可以衍生出很多的组合用法，不需要使用继承。

典型用法就是Dialog。

# API

## React

### 组件

* React.Component
* React.PureComponent
* React.memo

#### React.Component

##### 生命周期

* 创建
  * constructor()
  * static getDerivedStateFromProps()
  * render()
  * componentDidMount()
* 更新
  * static getDerivedStateFromProps()
  * shouldComponentUpdate()
  * render()
  * getSnapshotBeforeUpdate()
  * componentDidUpdate()
* 卸载
  * componentWillUnmount()
* 错误处理
  * static getDerivedStateFromError()
  * componentDidCatch()

##### 其它API

* setState()
* forceUpdate()

##### 类属性

* defaultProps
* displayName

##### 实例属性

* props
* state

### 创建元素

* createElement()
* createFactory()

### 操作元素

* cloneElement()
* isValidElement()
* React.Children

### Fragments

* React.Fragment

### Refs

* React.createRef
* React.forwardRef
