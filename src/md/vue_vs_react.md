# React VS Vue

## 生态

<font size=4 color=#a39f93>Vue</font>

- 核心：vue
- 路由：vue-route
- 状态树：vuex
- 服务端渲染：nuxt
- 脚手架：vue-cli3

<font size=4 color=#a39f93>React</font>

- 核心：react, react-dom
- 路由：react-router, react-router-dom
- 状态树：react-redux, redux
- 服务端渲染：next
- 脚手架：create-react-app

## 组件实例

<br/>
<div align=center><font color=green size=5>2.1 基础实例</font></div>
<br/>

<font size=4 color=#a39f93>Vue2.0基本模板</font>

- Vue模板编译
- 模块式写法

```javascript
<template></template>

<script>
export default {
  name: '',
  props: [],
  component: {},
  data() {
    return {}
  },
  computed: {},
  watch() {},
  methods: {}
  // ...生命周期等
}
</script>

<style></style>
```

<br/>
<font size=4 color=#a39f93>React 基本模板</font>
<br/>
<br/>

- jsx语法
- class声明


```javascript
import React from 'react'

class Comp extends React.Component {
  constructor(props) {
    super(props)
    this.state = {}
  }
  ...
  (methods or lifestyle)
  render() {return {}}
}
```


<br/>
<div align=center><font color=green size=5>2.2 生命周期</font></div>
<br/>

<font size=4 color=#a39f93>Vue2.0</font>

- beforeCreated。服务端渲染也拥有
- created。 可以访问this，服务端渲染也拥有
- beforeMounted
- mounted。可以访问DOM
- beforeUpdate
- updated
- beforeDestory
- destory
- errorCaptured 错误捕获
- keep-alive的activated和deactivated
- serverPrefetch: 服务端渲染使用，和asyncData一样用于返回数据，返回一个promise

<br/>
<font size=4 color=#a39f93>React</font>
<br/>
<br/>

- getDefaultProps。服务端渲染也拥有
- getInitialState。服务端渲染也拥有
- componentWillMount。服务端渲染也拥有
- render。服务端渲染也拥有
- componentDidMount
- componentWillReceiveProps(nextprops)
- shouldComponentUpdate。 return false 可以阻止渲染
- componentWillUpdate(nextProps, nextState)
- componentDidUpdate
- componentWillUnmount

<br/>
<font size=4 color=#a39f93>React17</font>
<br/>
<br/>

React17 废弃3个 will的生命周期:
- componentWillMount
- componentWillRecieveProps
- componentWIllUpdate

简单来说就是这三个生命周期函数容易被误解并滥用，可能会对异步渲染造成潜在的问题。
可用UNSAFE_xxx 来取消eslint的报错。

新增三个生命周期
- getDerivedStateFromProps(nextProps, prevState) 

用于替换componentWillReceiveProps，可以用来控制 props 更新 state 的过程；它返回一个对象表示新的 state；如果不需要更新，返回 null 即可

- getSnapshotBeforeUpdate(nextProps, prevState) 。用于替换componentWillUpdate
- componendDidCatch(error, info)。 新增，用于捕捉错误


<br/>
<div align=center><font color=green size=5>2.3 访问DOM</font></div>
<br/>

<font size=4 color=#a39f93>Vue2.0</font>

- 虚拟节点上的ref搭配this.\$refs(原生DOM的this.\$refs.xxx 或 组件的this.\$refs.xxx.$el)
- document原生API

<br/>
<font size=4 color=#a39f93>React</font>
<br/>
<br/>

- this.getDOMNode()。 14后移除
- 虚拟节点上的ref 搭配this.refs
- ReactDOM.findDomNode(this.refs.xxx)

*注意：组件上使用refs获取到的都将是组件的实例*

<br/>
<div align=center><font color=green size=5>2.4 访问实例</font></div>
<br/>

<font size=4 color=#a39f93>Vue2.0</font>

- template模板上可直接访问
- js里只要正常使用，不使用箭头函数,this.xxx 可代理访问this.data.xxx 

```javascript
computed: {
  this.xxx
}
watch() {
  name() {
    this.xxx
  }
}
methods: {
  methods() {
    this.xxx
  }
}
```

<br/>
<font size=4 color=#a39f93>React</font>
<br/>
<br/>

- this.state.xxx 获取数据
- 使用方法有两种方式：1，使用bind；2，使用箭头函数
  
```javascript
class MyComp extends React.Component {
  constructor(props) {
    super(props)
    this.state = {
      name: 'zty'
    }
    // 箭头函数代码少些，bind的性能稍微好些
    this.handleRegister = this.handleRegister.bind(this)
  }
  handleCLick1() {
    this.props.register(this.props.state)
  }
  handleCLick2() {
    this.props.register(this.props.state)
  }
  render() {
    return (
      <div>
        <p onClick={this.handleClick1}></p>
        <p onClick={() => this.handleClick2()}></p>
      </div>
    )
  }
}
```

<br/>
<div align=center><font color=green size=5>2.5 通信机制</font></div>
<br/>

<font size=4 color=#a39f93>Vue2.0</font>

- props 和 $emit     
- \$parent 和 \$children
- ref 和 refs
- \$attr 和 \$listener：v-bind="\$attrs" v-on="\$listeners"
- eventBus：通过eventBus.$emit派发 eventBus.\$on监听 实际就是实现事件的发布订阅，但不知道来源哪里不推荐使用
  
```javascript
// eventBus.js
const eventBus = new Vue()
export default eventBus
vuex
```
- Provider 和 inject
- vuex

<br/>
<font size=4 color=#a39f93>React</font>
<br/>
<br/>

- props
- 
 ```javascript
// eventBus.js
// 父组件
<child name="11" cb={this.callback}></child>

// 子组件
this.props.name // 获取属性
this.props.cb() // 向父组件通信
```
- context。 可跨级通信，但不知道来源哪里不推荐使用。基于生产者消费者模式

```javascript
// 父组件
class ParentComponent extends React.Component {
  // 声明Context对象属性
  static childContextTypes = {
    propA: PropTypes.string,
    methodA: PropTypes.func
  }
  
  // 返回Context对象，方法名是约定好的
  getChildContext () {
    return {
      propA: 'propA',
      methodA: () => 'methodA'
    }
  }
}

// 子组件
class ChildComponent extends React.Component {
  // 声明需要使用的Context属性
  static contextTypes = {
    propA: PropTypes.string
  }
  
  render () {
    const {
      propA,
      methodA
    } = this.context
    
    return ...
  }
}
```

-redux和react-redux
- 用js实现发布订阅模式

<br/>
<font size=4 color=#a39f93>React17</font>
<br/>
<br/>

React17 会废弃childContext 使用新API - createContext()

```javascript
const ThemeContext = React.createContext('light'); // 提供默认值
class App extends React.Component {
  render() {
    return (
      <ThemeContext.Provider value="dark">
        <Toolbar />
      </ThemeContext.Provider>
    );
  }
}

// 后代组件 
class ThemedButton extends React.Component {
  render() {
    return (
      <ThemeContext.Consumer>
        // value将获取父组件提供的value值
        {value =>
          <div></div>
        }
      </ThemeContext.Consumer>
    )
  }
}
```

<br/>
<div align=center><font color=green size=5>2.6 Props校验</font></div>
<br/>

<font size=4 color=#a39f93>Vue2.0</font>

```javascript
export default {
  props: {
    name: {
      type: String,
      required: true，
      default："zty",
      validator: function(val) {return !!val}
    }
  }
}
```

<br/>
<font size=4 color=#a39f93>React</font>
<br/>
<br/>

React  React v15.5 版本后将props检验移到了 prop-types 库

```javascript
class MyComponent extends React.Component {
  static propTypes = {
    name: PropTypes.string,
    children: PropTypes.element.isRequired
  };
  render() {
  // ... do things with the props
  }
}
```

## 路由

## 状态树


## 结束语

更新中...