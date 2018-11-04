## StrictMode

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