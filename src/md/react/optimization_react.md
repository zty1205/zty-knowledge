# React 相关优化

## 更合理的Props和state写法

```JSX
function(props) {
  return <div>{props.name}</div>
}
class App extends React.Component {
  constructor(props) {
    super(props)
    this.state = {
      user: 'zty',
      age: 18
    }
    this.click = this.click.bind(this) // 绑定函数方式1
    this.data = {user: 'zty'}
  }
  click() {}
  click1() {}
  render() {
    return (
      <div>
        <button onClick={this.click1.bind(this)}></button> // 绑定函数方式2
        <button onClick={() => this.click1()}></button> // 绑定函数方式3
        <button style={{color: 'red'}}></button> // 内联对象
        <btn {...this.state}></btn> // 传递了不必要的属性
      </div>
    )
  }
}
```

在我们的render函数不推荐使用bind和箭头函数，因为每次render的时候都需要重新bind和生成click事件的响应函数，内联的对象也是如此。

推荐:
- 在构造函数里使用bind
- 尽量不使用内联的对象
- 不传递不必要的属性


## 合理使用shouldComponentUpdate 和 PureComponent

react的生命周期 shouldComponentUpdate 可以控制本次是否重新渲染组件。当父组件更新的时候
我们可以进行props或state的对比来判断本次更新数据是否渲染组件。你可以使用以下方式:

- 写一个简单的对象对比函数
- 使用immuntable.js
- 使用React.PureComponent

```javascript
shouldComponentUpdate(nextProps, nextState) {
  return true || false
}
```