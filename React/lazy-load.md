## lazy-load

避免打包代码体积太大，使用webpack分割代码打包成多个文件，在运行时动态加载。

### import()

使用import()语法，webpack遇到这种语法会自动分割代码。

```js
import("./math").then(math => {
  console.log(math.add(16, 26));
});
```

### React.lazy

React.lazy方法允许将动态导入呈现为常规组件。

```js
const OtherComponent = React.lazy(() => import('./OtherComponent'));

function MyComponent() {
  return (
    <div>
      <OtherComponent />
    </div>
  );
}
```

React.lazy参数内必须调用动态import()，返回Promise，Promise中解析一个模块，这个模块带有default导出的React组件。

#### Suspense

在React.lazy导入的组件还未加载时，使用Suspense呈现替代内容，如加载框。

```js
const OtherComponent = React.lazy(() => import('./OtherComponent'));

function MyComponent() {
  return (
    <div>
      <Suspense fallback={<div>Loading...</div>}>
        <OtherComponent />
      </Suspense>
    </div>
  );
}
```

fallback接受React元素。Suspense可以在懒加载组件之上的任意位置，也可以包含多个懒加载组件。

#### 错误边界

当模块加载失败时，使用错误边界处理错误。错误边界位于懒加载组件之上任意位置。

```js
import MyErrorBoundary from '/MyErrorBoundary';
const OtherComponent = React.lazy(() => import('./OtherComponent'));
const AnotherComponent = React.lazy(() => import('./AnotherComponent'));

const MyComponent = () => (
  <div>
    <MyErrorBoundary>
      <Suspense fallback={<div>Loading...</div>}>
        <section>
          <OtherComponent />
          <AnotherComponent />
        </section>
      </Suspense>
    </MyErrorBoundary>
  </div>
);
```

### 基于路由的代码分割

```js
import { BrowserRouter as Router, Route, Switch } from 'react-router-dom';
import React, { Suspense, lazy } from 'react';

const Home = lazy(() => import('./routes/Home'));
const About = lazy(() => import('./routes/About'));

const App = () => (
  <Router>
    <Suspense fallback={<div>Loading...</div>}>
      <Switch>
        <Route exact path="/" component={Home}/>
        <Route path="/about" component={About}/>
      </Switch>
    </Suspense>
  </Router>
);
```

### 注意事项

React.lazy只支持default导出。如果需要使用命名导出的组件，创建一个模块使用default导出。

```js
// ManyComponents.js
export const MyComponent = /* ... */;
export const MyUnusedComponent = /* ... */;

// MyComponent.js
export { MyComponent as default } from "./ManyComponents.js";

// MyApp.js
import React, { lazy } from 'react';
const MyComponent = lazy(() => import("./MyComponent.js"));
```
