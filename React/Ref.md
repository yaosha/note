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
