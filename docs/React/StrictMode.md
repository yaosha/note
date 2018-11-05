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
