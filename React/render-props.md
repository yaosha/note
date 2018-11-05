## render-props

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
