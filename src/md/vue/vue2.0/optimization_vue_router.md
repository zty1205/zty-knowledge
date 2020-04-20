# Vue-Router相关优化

## 异步加载路由

异步加载路由对应的组件：异步打包chunk,在路由跳转该页面是加载该chunk
- import()
- require.ensure()

例子：

```javascript
const router = new VueRouter({
  routes: [
    {
      path: '/foo',
      component: import(/* webpackChunkName: "group-foo" */ './Foo.vue')
    }
  ]
})
```


## 自动生成路由文件

通过require.context，自动生成路由 规范命名，可参考ssr的命名，可以实现动态路由，即:id.vue（nuxt用的是glob）。

require.context返回的是__webpack__require__(idx)  '../components/foo'对应的下标


```javascript
let r = require.context('../components/foo', true, /\.vue$/)
let arr = []
r.keys().map(name => {
  const nameArr = name.split('.')
  const comp = r(name).default
  arr.push({
    path: `/foo${nameArr[1]}`,
    component: comp,
    title: comp.name,
    meta: comp.meta
  })
});
console.log('arr = ', arr)
```


