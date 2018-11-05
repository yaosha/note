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
