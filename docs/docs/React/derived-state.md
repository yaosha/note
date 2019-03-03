
## dont-need-derived-state

`derived-state`是从props衍生的state，只能在`getDerivedStateFromProps`中使用，这一章节其实就是讲`getDerivedStateFromProps`的用法。

### 使用场景

只能在`getDerivedStateFromProps`中使用，唯一目的是使组件根据props变更的结果修改内部state。

`getDerivedStateFromProps`必须谨慎使用。

所有根据props变更的结果修改内部state的问题最终都可以简化为2个问题：
* 无条件从props更新state
* 在props和state不匹配时更新state

以上2个问题都会导致频繁重置state，而且如果衍生state只是记录根据props计算的值，考虑`memoization`。

### 使用getDerivedStateFromProps的常见错误

* controlled：作为props传入的数据
* uncontrolled：仅存在于state的数据

最常见的错误是混合`controlled`和`uncontrolled`，当衍生state也可以被`setState`变更时，数据没有单一的真实来源。

#### 无条件的将props复制到state

常见误解：仅当props更改时才会调用`getDerivedStateFromProps`和`componentWillReceiveProps`。事实是无论props是否与以前不同，这些生命周期都会在父组件重新渲染时被调用。因此，使用在这些生命周期中无条件地覆盖state始终是不安全的。这样做会导致state更新丢失。

示例：无条件的将props复制到state
```js
class EmailInput extends Component {
  state = { email: this.props.email };

  render() {
    return <input onChange={this.handleChange} value={this.state.email} />;
  }

  handleChange = event => {
    this.setState({ email: event.target.value });
  };

  componentWillReceiveProps(nextProps) {
    // This will erase any local state updates!
    // Do not do this.
    this.setState({ email: nextProps.email });
  }
}
```

EmailInput在父组件重渲染时，input中所有输入的内容会丢失，因为componentWillReceiveProps中代码将state重置为props的值。即使我们在重置之前比较`nextProps.email !== this.state.email`，也是如此。

增加`shouldComponentUpdate`仅在this.props.email已更改时才能重新渲染可以解决这个问题。但是实际上，组件一般有多个props，其它props变更还是会导致重绘和不正确的重置。函数和对象props经常使用行内创建，这使得很难实现一个只有在发生props更改时才能可靠地返回true的`shouldComponentUpdate`。所以，`shouldComponentUpdate`最好用于性能优化，而不是确保衍生state的正确性。

所以无条件的将props复制到state不是一个好主意。

### props变更时清除state

继续上面的例子，我们可以通过仅在props.email更改时更新它来避免意外删除state：
```js
class EmailInput extends Component {
  state = {
    email: this.props.email
  };

  componentWillReceiveProps(nextProps) {
    // Any time props.email changes, update state.
    if (nextProps.email !== this.props.email) {
      this.setState({
        email: nextProps.email
      });
    }
  }
  
  // ...
}
```

虽然只在props变更时才会删除state，但是还是有一个问题。有一种场景，需要在props.email相同时也重置state，比如在使用相同email的不同账号间切换时，email无法变更。

这种设计存在根本缺陷，也是一个容易犯的错误。

解决问题的关键是对于任何数据，必须选择一个拥有它的单一组件作为真实来源，并避免在其它组件中复制它。

### 首选解决方案

#### 建议：fully-controlled-component

一种避免以上问题的方式是将state完全从组件移出。在父级表单组件手动控制输入值。

```js
function EmailInput(props) {
  return <input onChange={props.onChange} value={props.email} />;
}
```

#### 建议：fully-uncontrolled-component-with-a-key

另一种选择是组件完全拥有state。在这种情况下，我们的组件仍然可以接受初始值的prop，但它会忽略对该prop的后续更改：

```js
class EmailInput extends Component {
  state = { email: this.props.defaultEmail };

  handleChange = event => {
    this.setState({ email: event.target.value });
  };

  render() {
    return <input onChange={this.handleChange} value={this.state.email} />;
  }
}
```

为了重置输入框的值，可以使用名为`key`的React特殊props。当`key`变更时，React会创建新的组件实例而不是更新组件。不需要将key用在每个输入组件，在表单设置key更有用。

```js
<EmailInput
  defaultEmail={this.props.user.email}
  key={this.props.user.id}
/>
```
>注意
>
>虽然这可能看起来很慢，但性能差异通常是微不足道的。 如果组件在更新时运行复杂的逻辑，则使用key甚至可以更快，因为对于该子树绕过了差异。

#### 备选方案1：使用ID props重置uncontrolled-component

如果key由于某种原因不起作用（初始化组件可能非常昂贵），那么可行但麻烦的解决方案是在getDerivedStateFromProps中监视“userID”的更改：

```js
class EmailInput extends Component {
  state = {
    email: this.props.defaultEmail,
    prevPropsUserID: this.props.userID
  };

  static getDerivedStateFromProps(props, state) {
    // Any time the current user changes,
    // Reset any parts of state that are tied to that user.
    // In this simple example, that's just the email.
    if (props.userID !== state.prevPropsUserID) {
      return {
        prevPropsUserID: props.userID,
        email: props.defaultEmail
      };
    }
    return null;
  }

  // ...
}
```

#### 备选方案2：使用实例方法重置uncontrolled-component

更少见的是，没有适当的值可以用作`key`，但是需要重置状态。 一种解决方案是每次要重置时将`key`重置为随机值或自动递增数字。 另一个可行的替代方法是公开实例方法以强制重置内部state：

```js
class EmailInput extends Component {
  state = {
    email: this.props.defaultEmail
  };

  resetEmailForNewUser(newEmail) {
    this.setState({ email: newEmail });
  }

  // ...
}
```

#### 总结

总而言之，在设计组件时，重要的是确定其数据是受控还是不受控制。

不要试图在state中拷贝props，而是使用`controlled-component`，将state提升到父组件处理。 例如，比起子组件接受“已提交”的props.value并跟踪“草稿”state.value，而是让父组件管理state.draftValue和state.committedValue并直接控制子项的值。 这使得数据流更加明确和可预测。

对于`uncontrolled-component`，如果在特定props（通常是ID）发生变化时尝试重置state，则可以选择以下几种方法：

建议：使用key属性重置所有内部state。
备选方案1：在`getDerivedStateFromProps`监听特定props（例如props.userID）的变更，仅重置某些state。
备选方案2：使用ref调用实例方法重置state。

### memoization

使用衍生state确定用于`render`的昂贵的值，且值仅在输入改变时重新计算，这个技巧称为`memoization`。

使用衍生state进行记忆并不一定是坏事，但它通常不是最佳解决方案。管理衍生state存在固有的复杂性，并且这种复杂性随着每个附加props而增加。 例如，如果向组件状态添加第二个衍生state，那么需要单独跟踪对两者的更改。

示例：搜索查询匹配的项目，使用衍生state来存储过滤后的列表
```js
class Example extends Component {
  state = {
    filterText: "",
  };

  // *******************************************************
  // 注意: 这个示例的不是推荐的方法
  // 看下面的示例获取更好的建议
  // *******************************************************

  static getDerivedStateFromProps(props, state) {
    // Re-run the filter whenever the list array or filter text change.
    // Note we need to store prevPropsList and prevFilterText to detect changes.
    if (
      props.list !== state.prevPropsList ||
      state.prevFilterText !== state.filterText
    ) {
      return {
        prevPropsList: props.list,
        prevFilterText: state.filterText,
        filteredList: props.list.filter(item => item.text.includes(state.filterText))
      };
    }
    return null;
  }

  handleChange = event => {
    this.setState({ filterText: event.target.value });
  };

  render() {
    return (
      <Fragment>
        <input onChange={this.handleChange} value={this.state.filterText} />
        <ul>{this.state.filteredList.map(item => <li key={item.id}>{item.text}</li>)}</ul>
      </Fragment>
    );
  }
}
```

此实现太复杂，因为它必须单独跟踪和检测props和state的变化，以便正确更新过滤后的列表。 在下面示例中，我们可以通过使用PureComponent并将过滤操作移到render方法来简化操作：

```js
// PureComponents only rerender if at least one state or prop value changes.
// Change is determined by doing a shallow comparison of state and prop keys.
class Example extends PureComponent {
  // State only needs to hold the current filter text value:
  state = {
    filterText: ""
  };

  handleChange = event => {
    this.setState({ filterText: event.target.value });
  };

  render() {
    // The render method on this PureComponent is called only if
    // props.list or state.filterText has changed.
    const filteredList = this.props.list.filter(
      item => item.text.includes(this.state.filterText)
    )

    return (
      <Fragment>
        <input onChange={this.handleChange} value={this.state.filterText} />
        <ul>{filteredList.map(item => <li key={item.id}>{item.text}</li>)}</ul>
      </Fragment>
    );
  }
}
```

上面示例也有缺陷，对于大型列表过滤会很慢，而且在其它props变更时`PureComponent`不会阻止重绘。为了解决这两个问题，可以添加一个memoization帮助器，以避免不必要地重新过滤列表：

```js
import memoize from "memoize-one";

class Example extends Component {
  // State only needs to hold the current filter text value:
  state = { filterText: "" };

  // Re-run the filter whenever the list array or filter text changes:
  filter = memoize(
    (list, filterText) => list.filter(item => item.text.includes(filterText))
  );

  handleChange = event => {
    this.setState({ filterText: event.target.value });
  };

  render() {
    // Calculate the latest filtered list. If these arguments haven't changed
    // since the last render, `memoize-one` will reuse the last return value.
    const filteredList = this.filter(this.props.list, this.state.filterText);

    return (
      <Fragment>
        <input onChange={this.handleChange} value={this.state.filterText} />
        <ul>{filteredList.map(item => <li key={item.id}>{item.text}</li>)}</ul>
      </Fragment>
    );
  }
}
```

使用`memoization`约束：

* 将memoized函数附加到组件实例，防止组件的多个实例重置彼此的memoized密钥。
* 使用可以限制缓存大小的memoization，以防止内存泄漏。 （在上面的例子中，我们使用了memoize-one，因为它只缓存了最新的参数和结果。）
* 如果每次父组件呈现时都重新创建props.list，则本节中显示的任何实现都不起作用。 但在大多数情况下，这种设置是合适的。
