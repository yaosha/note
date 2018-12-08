## Context

Context可以视作全局变量，用于跨组件传递数据。
之前版本遗留的context API在v16中支持，在下个大版本移除。

### API

#### React.createContext

创建context对象，React组件通过匹配最近层级的Provider读取这个context，defaultValue用于在组件找不到匹配的Provider的情况。

```js
const myContext = React.createContext(defaultValue);
```

#### Context.Provider

value作为传递给Provider的子组件的值。当provider的value改变时，所有子组件都会重新渲染。不受shouldComponentUpdate的影响。使用Object.is比较value的值。

```html
<MyContext.Provider value={/* some value */}>
```

#### Class.contextType

将context对象赋值给组件类的contextType属性，组件内部可以使用this.context取得最近层级Context对象的当前值。

示例1：单个context
```js
class MyClass extends React.Component {
  componentDidMount() {
    let value = this.context;
    /* perform a side-effect at mount using the value of MyContext */
  }
  componentDidUpdate() {
    let value = this.context;
    /* ... */
  }
  componentWillUnmount() {
    let value = this.context;
    /* ... */
  }
  render() {
    let value = this.context;
    /* render something based on the value of MyContext */
  }
}
MyClass.contextType = MyContext;
```

示例2：单个context，static语法
```js
class MyClass extends React.Component {
  static contextType = MyContext;
  render() {
    let value = this.context;
    /* render something based on the value */
  }
}
```

示例3：多个context
```js
// Theme context, default to light theme
const ThemeContext = React.createContext('light');

// Signed-in user context
const UserContext = React.createContext({
  name: 'Guest',
});

class App extends React.Component {
  render() {
    const {signedInUser, theme} = this.props;

    // App component that provides initial context values
    // highlight-range{2-3,5-6}
    return (
      <ThemeContext.Provider value={theme}>
        <UserContext.Provider value={signedInUser}>
          <Layout />
        </UserContext.Provider>
      </ThemeContext.Provider>
    );
  }
}

function Layout() {
  return (
    <div>
      <Sidebar />
      <Content />
    </div>
  );
}

// A component may consume multiple contexts
function Content() {
  // highlight-range{2-10}
  return (
    <ThemeContext.Consumer>
      {theme => (
        <UserContext.Consumer>
          {user => (
            <ProfilePage user={user} theme={theme} />
          )}
        </UserContext.Consumer>
      )}
    </ThemeContext.Consumer>
  );
}
```

如果有多个经常一起使用的context，可以合并成一个。

#### Context.Consumer

```html
<MyContext.Consumer>
  {value => /* render something based on the context value */}
</MyContext.Consumer>
```

订阅context变化的React组件，可以在函数式组件内部订阅context。

### 注意事项

使用对象作为provider的value时，必须使用变量。不可以直接赋值{…}，因为每次都是新对象，会触发render。
