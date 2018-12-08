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
