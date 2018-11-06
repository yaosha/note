## Fragment

React.Fragment是一个内置组件，用于返回多个元素但是不增加额外的DOM。

```js
render() {
  return (
    <React.Fragment>
      <ChildA />
      <ChildB />
      <ChildC />
    </React.Fragment>
  );
}
```

界面实际上展示的：
```html
<ChildA />
<ChildB />
<ChildC />
```

唯一可使用的属性是key，用于在列表循环中使用。
```js
function Glossary(props) {
  return (
    <dl>
      {props.items.map(item => (
        // Without the `key`, React will fire a key warning
        <React.Fragment key={item.id}>
          <dt>{item.term}</dt>
          <dd>{item.description}</dd>
        </React.Fragment>
      ))}
    </dl>
  );
}
```
