




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
    const style = { position: 'absolute', left: mouse.x, top: mouse.y }；
    return (
      <img src="/cat.jpg" style={style} />
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
    const style = { height: '100%' };
    return (
      <div style={style} onMouseMove={this.handleMouseMove}>

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

#### React.PureComponent

`React.PureComponent`和`React.Component`类似，不同之处在于`React.Component`没有实现`shouldComponentUpdate()`，但是`React.PureComponent`使用浅层比较props和state实现`shouldComponentUpdate()`。

#### React.memo

```js
const MyComponent = React.memo(function MyComponent(props) {
  /* render using props */
});
```

`React.memo`是一个高阶组件，和`React.PureComponent`类似，但是只用于函数式组件。

如果函数组件对相同的props返回一样的结果，则可以使用`React.memo`缓存返回结果以提高性能。这意味着React对相同的props跳过渲染组件，使用上次渲染的结果。

默认浅层比较props对象，如果需要控制比较过程，传递自定义比较函数作为第二个参数。

```js
function MyComponent(props) {
  /* render using props */
}
function areEqual(prevProps, nextProps) {
  /*
  return true if passing nextProps to render would return
  the same result as passing prevProps to render,
  otherwise return false
  */
}
export default React.memo(MyComponent, areEqual);
```

### 创建元素

* createElement()
* createFactory()

#### createElement()

```js
React.createElement(
  type,
  [props],
  [...children]
)
```

根据给定type创建并返回React元素。

* type
  * 字符串：HTML标签名称，例如'div','span'
  * 组件：自定义组件名称变量或React.fragment

#### createFactory()

这个方法是遗留的，禁止使用。

### 操作元素

* cloneElement()
* isValidElement()
* React.Children

#### cloneElement()

```js
React.cloneElement(
  element,
  [props],
  [...children]
)
```

使用element作为起点克隆并返回一个新的React元素。结果元素具有原始元素的props和新的props浅层合并的结果。新的children会替换之前的children。key和ref保留，意味着ref指向新的元素。

几乎等价于：
```js
<element.type {...element.props} {...props}>{children}</element.type>
```

#### isValidElement()

```js
React.isValidElement(object)
```
验证对象是否为React元素，返回true或false。

#### React.Children

`React.Children`提供了处理this.props.children不透明数据结构的方法。

#### React.Children.map

```js
React.Children.map(children, function[(thisArg)])
```
对children的每个直接子元素调用函数，函数this指向传入的thisArg。如果children为null或undefined，方法返回null或undefined。

#### React.Children.forEach

```js
React.Children.forEach(children, function[(thisArg)])
```
和`React.Children.map`类似，但是不返回数组。

#### React.Children.count

```js
React.Children.count(children)
```
返回children内组件数量，等价于map或forEach回调函数执行次数。

#### React.Children.only

```js
React.Children.only(children)
```
验证children只包含一个子元素并返回该元素，否则跑出错误。

#### React.Children.toArray

```js
React.Children.toArray(children)
```
将children不透明数据结构作为平面数组返回，每个子项分配key。

### Fragments

* React.Fragment

### Refs

* React.createRef：创建ref，可通过ref属性传给组件
* React.forwardRef：创建React组件，将ref转发到下层组件。

```js
// highlight-range{1-2}
const FancyButton = React.forwardRef((props, ref) => (
  <button ref={ref} className="FancyButton">
    {props.children}
  </button>
));

// You can now get a ref directly to the DOM button:
const ref = React.createRef();
<FancyButton ref={ref}>Click me!</FancyButton>;
```

## React.Component

`React.Component`是使用ES6类语法定义组件的基类。

### 生命周期

![image](React生命周期v16.4.png)

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

#### render()

```js
render()
```

render是组件内唯一需要实现的方法，其它方法按需实现。

* 返回值
  * React元素：JSX语句
  * 数组或Fragment：返回多个元素
  * Portal：将children渲染到不同的DOM子树
  * 字符串或数字：作为文本节点DOM渲染
  * boolean或null：什么也不渲染
* 必须是纯粹函数：不修改state、每次调用返回相同结果、不直接和浏览器交互，如果必须和浏览器交互，在`componentDidMount()`或其它生命周期处理。

#### constructor()

```js
constructor(props)
```

* 调用时机：组件渲染之前
* 用途
  * 初始化state，constructor是唯一可以对`this.state`直接赋值的地方
  * 绑定方法到实例
* 使用限制
  * 必须在其它任何语句之前调用`super(props)`
  * 禁止调用`setState`方法
  * 避免在构造函数中引入任何副作用或订阅。 对于这些用例，请改用componentDidMount（）。
* 常见错误
  * 避免将props复制到state

```js
constructor(props) {
  super(props);
  // Don't call this.setState() here!
  this.state = { counter: 0 };
  this.handleClick = this.handleClick.bind(this);
}
```

#### componentDidMount()

```js
componentDidMount()
```

* 调用时机：组件渲染之后立即执行
* 用途
  * 初始化DOM节点
  * 请求远程端点数据
  * 订阅，记得在`componentWillUnmount()`中取消订阅
  * 调用`setState()`
    * 结果：触发重渲染，但是发生在浏览器更新屏幕之前，保证即时`render()`触发两次，用户也不会看到中间过程
    * 注意：只在必要时谨慎使用，会引起性能问题

#### componentDidUpdate()

```js
componentDidUpdate(prevProps, prevState, snapshot)
```

* 调用时机：组件更新后立即执行，不会再初次render时调用
* 用途
  * 操作DOM节点
  * 比较当前和上次props，确定是否请求远程端点数据
  * 调用`setState()`
    * 结果：触发重渲染，中间过程用户不可见
    * 注意：必须包含条件，否则导致无限循环；导致性能问题

```js
componentDidUpdate(prevProps) {
  // Typical usage (don't forget to compare props):
  if (this.props.userID !== prevProps.userID) {
    this.fetchData(this.props.userID);
  }
}
```

如果组件实现`getSnapshotBeforeUpdate()`，返回值将作为`componentDidUpdate()`的第三个参数，否则这个参数为undefined。

#### componentWillUnmount()

```js
componentWillUnmount()
```

* 调用时机：组件卸载和销毁之前立即执行
* 用途：执行清理，例如使计时器无效、取消网络请求、清除在`componentDidMount()`创建的订阅
* 注意：不要调用`setState()`

#### shouldComponentUpdate()

```js
shouldComponentUpdate(nextProps, nextState)
```

* 调用时机：接收新的state或props时，组件渲染前执行，在初次渲染或`forceUpdate()`调用时不执行
* 目的：性能优化
* 默认行为：任何state或props变更，触发重渲染
* 用途：指定组件的输出是否被state或props的改变影响
* 注意
  * 不要依赖这个方法阻止渲染，使用前考虑是否可以使用`React.PureComponent`代替
  * 如果确定使用，比较`this.props`和`nextProps`、`this.state`和`nextState`，返回false告诉React跳过更新。但是当state变更时，即使返回false也不会阻止组件重渲染。
  * 不建议使用深度比较或`JSON.stringify()`，效率太低，降低性能
* 对其它生命周期函数影响：返回false，`UNSAFE_componentWillUpdate()`, `render()`, 和 `componentDidUpdate()`不会执行

#### static getDerivedStateFromProps()

```js
static getDerivedStateFromProps(props, state)
```

* 调用时机：render之前调用，无论初次加载还是随后的更新，无论props还是state变更。
* 返回值：返回对象更新state，或null什么也不更新
* 使用场景：state依赖props随时间的变化。
* 缺点：导致冗长的代码，并使组件难以维护，考虑替代方案
  * 执行副作用（动画或拉取数据）响应props变更，改用`componentDidUpdate`替代
  * 当props变更时重新计算某些数据，使用memoization helper
  * 当props变更时重置state，使用fully-controlled-component或fully-uncontrolled-component-with-a-key
* 此方法是静态方法，无法访问组件实例。可以通过在类定义之外提取组件props和state的纯函数，在getDerivedStateFromProps（）和其他类方法之间重用一些代码。

### 其它API

* setState()
* forceUpdate()

### 类属性

* defaultProps
* displayName

### 实例属性

* props
* state
