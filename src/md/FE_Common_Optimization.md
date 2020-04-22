# 前端常见优化

## 构建工具帮我们完成的

- 前置css，后置js，防止js加载，运行影响页面渲染
- 将小图达成base64，减少资源请求。[file-loader, url-loader,...]
- 压缩精简html，css和js，减小打包体积。 [uglifyjs, OptimizeCssAssetsPlugin, ...]
- Gzip压缩，该功能需要服务器支持才能正常显示页面
- css预处理器，开启css编程之路

## HTML
- 减少DOM数量
- 避免重排和重绘: 减少DOM操作，动画优先使用 ***opacity， transform*** 属性; 合并DOM的读写操作，如使用 document.createDocumentFragment(); 使用虚拟DOM的思想
- 使用特殊的函数，优化条件渲染：window.requestAnimationFrame()， window.requestIdleCallback()

## CSS
- 避免使用css表达式
- 使用css sprite 雪碧图，减少图片请求
- 在不影响画质的情况下，使用合理的图片格式和压缩图片，优先使用JPG格式，如果能用css3实现动画，则尽量不使用GIF。如果能使用canvas或SVG实现，则尽量不使用图片

## JS
- 使用 JavaScript Cache API，我们可以使用 service worker。
- 延迟不必要的 JS 首屏加载 defer , aysc, 动态添加script节点
- 删除未使用的 JavaScript 和 合并重复的代码 减少编译时间（JIT）
- 避免内存泄漏 意外的全局变量；没有销毁的计时器；已经删除的 DOM 还是被引用，（删除DOM后将变量设值为 null 可以避免这个问题）
- 避免使用全局变量 & 优先使用局部变量，作用域链查找更快
- 使用 web workers 处理需要大量执行时间的代码（子线程）
- 合理使用事件代理。合并类似的操作，节约内存空间，减少 DOM 操作
- 使用高级函数等，例如addEvent的兼容惰性加载函数
  
```javascript
let addEvent1 = (type, element, fun) => {
  if (element.addEventListener) {
    addEvent1 = (type, element, fun) => {
      element.addEventListener(type, fun, false);
    }
  } else if (element.attachEvent) {
    addEvent1 =  (type, element, fun) => {
      element.attachEvent('on' + type, fun);
    }
  } else {
    addEvent1 = (type, element, fun) => {
      element['on' + type] = fun;
    }
  }
  return addEvent1(type, element, fun);
}
```

## 框架中的优化

<a href="./vue/vue2.0.md">vue相关优化</a>

<a href="./react/react.md">react相关优化</a>

## 工程化
- 使用CDN分发网络，请求资源更快
- 减少HTTP请求次数，减少DNS查询次数（尽量减少主机名），避免重定向
- DNS预获取 link标签 ref='dns-prefetch' herf=''
- 使AJAX可缓存：get请求可在客户端缓存；post请求不能再客户端缓存，但是服务端可以缓存数据（redis，memorycache等），提高请求速度。
- 模块化，组件库，工具库
- 微前端
- SSR和预渲染，提高渲染速度和更好的性能体验

## 结束语

持续更新中...