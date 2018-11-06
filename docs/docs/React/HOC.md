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
