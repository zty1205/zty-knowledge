# Vuex相关优化

## 动态注册store

store有个API：

```javascript
/**
 * 
 * @param {*} name 模块名
 * @param {*} module store中的一个模块
 * @param {*} config 配置，合并策略
 */
function registerModule(name, module, config) {}
```

我们可以开发一个插件：当某个页面需要使用vuex时，动态注册store。例子如下：

```javascript
export const dynamicModule = {
  install: function(Vue) {
    console.log('install')
    Vue.mixin({
      beforeCreate: function() {
        if (this.$options.dynamicVuex) {
          import('./store/module/simple.store.js').then(module => { // or require.ensure
            console.log('then module = ', module)
            this.$store.registerModule('dyModule', module.default)
          })
        }
      },

    })
  },
  uninstall: function() {
    ...
  }
}
```