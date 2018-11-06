## react-dom

react-dom提供了特定于DOM的方法，可以在应用顶层使用，也可以React模型外进行操作。

提供的方法：
* render()
* hydrate()
* unmountComponentAtNode()
* findDOMNode()
* createPortal()

### render()

```js
ReactDOM.render(element, container[, callback])
```

* 用途：在DOM节点container中加载React组件element，再次执行更新组件
* 参数
  * element：React元素
  * container：DOM元素容器
  * callback：可选，组件加载或更新后执行的回调
* 返回值：组件引用（对于无状态组件是null），遗留，禁止使用，如果需要组件引用使用ref

>注意
>
>初次执行，container的任何元素被替换，之后的执行调用React的diff算法更新变更DOM。
>不修改容器节点（仅修改容器的子节点）。可以将组件插入现有DOM节点而不覆盖现有子节点。
>不要使用这个方法做服务端渲染，使用[hydrate()](#hydrate())

### hydrate()

```js
ReactDOM.hydrate(element, container[, callback])
```

和render()类似，但是用来呈现由ReactDOMServer渲染的HTML内容。React尝试将事件侦听器附加到现有标记。

### unmountComponentAtNode()

```js
ReactDOM.unmountComponentAtNode(container)
```

* 用途：删除DOM中已加载的React组件并清除事件处理器和state。如果container中没有加载过React组件，则调用次函数不执行任何操作。
* 返回值：true表示卸载完成，false表示没有组件需要卸载

### findDOMNode()

findDOMNode是用于访问底层DOM节点的转义填充。在大多数情况下，不鼓励使用此方法，因为它会刺穿组件抽象。已在StrictMode中弃用。

```js
ReactDOM.findDOMNode(component)
```

* 用途：知道组件对应的原生DOM元素
* 替代方案：在需要的组件添加ref props
* 返回值：组件对应的DOM元素
  * render为null或false：返回null
  * render字符串：返回包含对应值得文本DOM节点
  * render Fragment：返回与第一个非空子节点对应的DOM节点
* 限制
  * 仅适用于已加载的组件（即已放置在DOM中的组件），否则引发异常
  * 不能用于函数式组件

### createPortal()

```js
ReactDOM.createPortal(child, container)
```

创建portal，将child加载到DOM层级外的DOM节点。

### ReactDOMServer

`ReactDOMServer`将组件呈现为静态标记，通常在node服务器使用：

```js
// ES modules
import ReactDOMServer from 'react-dom/server';
// CommonJS
var ReactDOMServer = require('react-dom/server');
```

可同时用在服务端和浏览器的方法
* renderToString()
* renderToStaticMarkup()

以下方法依赖`stream`，仅能用在服务端
* renderToNodeStream()
* renderToStaticNodeStream()

#### renderToString()

```js
ReactDOMServer.renderToString(element)
```

* 用途：将React元素渲染为其初始HTML
* 返回值：HTML字符串
* 场景：在服务器生成HTML，在初始请求时向下发送标记
  * 加快页面加载速度
  * 允许搜索引擎抓取页面以进行搜索引擎优化

如果在已有服务端渲染标记的node服务器上调用ReactDOM.hydrate()，则React将保留它并仅附加事件处理程序，从而使您具有非常高性能的首次加载体验。

#### renderToStaticMarkup()

```js
ReactDOMServer.renderToStaticMarkup(element)
```

与renderToString类似，但这不会创建React在内部使用的额外DOM属性，例如data-reactroot。如果要将React用作简单的静态页面生成器，这很有用，因为剥离额外的属性可以节省一些字节。

如果计划在客户端上使用React使标记可交互，请不要使用此方法。相反，在服务器上使用renderToString，在客户端上使用ReactDOM.hydrate（）。

#### renderToNodeStream()

```js
ReactDOMServer.renderToNodeStream(element)
```

* 用途：将React元素渲染为其初始HTML
* 返回值：输出HTML字符串的以utf-8编码的可读流，此流的HTML输出与ReactDOMServer.renderToString将返回的完全相同。
* 场景：在服务器生成HTML，在初始请求时向下发送标记
  * 加快页面加载速度
  * 允许搜索引擎抓取页面以进行搜索引擎优化

如果在已有服务端渲染标记的node服务器上调用ReactDOM.hydrate()，则React将保留它并仅附加事件处理程序，从而使您具有非常高性能的首次加载体验。

#### renderToStaticNodeStream()

```js
ReactDOMServer.renderToStaticNodeStream(element)
```

与renderToNodeStream类似，但这不会创建React在内部使用的额外DOM属性，例如data-reactroot。如果要将React用作简单的静态页面生成器，这很有用，因为剥离额外的属性可以节省一些字节。

此流的HTML输出与ReactDOMServer.renderToStaticMarkup将返回的完全相同。

如果您计划在客户端上使用React使标记交互，请不要使用此方法。而是在服务器上使用renderToNodeStream，在客户端上使用ReactDOM.hydrate()。

从此方法返回的流将返回以utf-8编码的字节流。如果需要另一种编码的流，请查看像iconv-lite这样的项目，该项目提供用于转码文本的转换流。