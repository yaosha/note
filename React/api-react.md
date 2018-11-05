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
