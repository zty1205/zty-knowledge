# Webpack

## 简介

&emsp;&emsp;
本质上，webpack 是一个现代 JavaScript 应用程序的静态模块打包器(module bundler)。当 webpack 处理应用程序时，它会递归地构建一个依赖关系图(dependency graph)，其中包含应用程序需要的每个模块，然后将所有这些模块打包成一个或多个 bundle.【1】

<br/>

## 核心概念

<br/>

<font color=green>入口(entry)</font>

&emsp;&emsp;
指示 webpack 应该使用哪个模块，来作为构建其内部依赖图的开始。进入入口起点后，webpack 会找出有哪些模块和库是入口起点（直接和间接）依赖的。

&emsp;&emsp;
可以通过在 webpack 配置中配置 entry 属性，来指定一个入口起点（或多个入口起点）。默认值为 ./src。

<font color=green>输出(output)</font>

&emsp;&emsp;
output 属性告诉 webpack 在哪里输出它所创建的 bundles，以及如何命名这些文件，默认值为 ./dist。基本上，整个应用程序结构，都会被编译到你指定的输出路径的文件夹中。你可以通过在配置中指定一个 output 字段，来配置这些处理过程

<font color=green>loader</font>

&emsp;&emsp;
loader 让 webpack 能够去处理那些非 JavaScript 文件（webpack 自身只理解 JavaScript）。loader 可以将所有类型的文件转换为 webpack 能够处理的有效模块

在更高层面，在 webpack 的配置中 loader 有两个目标：
- test 属性，用于标识出应该被对应的 loader 进行转换的某个或某些文件。
- use 属性，表示进行转换时，应该使用哪个 loader。

<font color=green>插件(plugins)</font>

&emsp;&emsp;
插件的范围包括，从打包优化和压缩，一直到重新定义环境中的变量。插件接口功能极其强大，可以用来处理各种各样的任务。

&emsp;&emsp;
plugins需要暴露出一个class, 在new WebpackPlugin()的时候通过构造函数传入这个插件需要的参数，在webpack启动的时候会先实例化plugin再调用plugin的apply方法，插件需要在apply函数里监听webpack生命周期里的事件，做相应的处理

<font color=green>模式(mode)</font>

&emsp;&emsp;
通过选择 development 或 production 之中的一个，来设置 mode 参数，你可以启用相应模式下的 webpack 内置的优化

简单例子：

```javascript
// 多个入口
module.exports = {
  mode: 'production',
  entry: {
    index: ["./src/index.js"],
    main: ["./src/main.js"]
  },
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'js/[name].[hash:8].js'
  },
  module: {
    rules: [{
      test: /\.js$/, // 正则匹配文件名
      exclude: '/node_modules/', // 排除 
      use: ['babel-loader']
    }
  },
  plugins: [ // 插件
    new copyWebpackPlugin([{
      from: path.resolve(__dirname, 'public/static'), 
      to: path.resolve(__dirname, 'dist'),
      ignore: ['index.html']
  }])
}
```

<br/>

## 基本流程

<br/>

1. 解析shell和config中的配置项，用于激活webpack的加载项和插件
2. webpack初始化工作，包括构建compiler对象，初始化compiler的上下文，loader和file的输入输出环境
3. 解析入口js文件，通过对应的工厂方法创建模块，使用acron生成AST树并且遍历AST，处理require的module，如果依赖中包含依赖则遍历build module，在遍历过程中会根据文件类型和loader配置找出合适的loader用来对文件进行转换
4. 调用seal方法，封装，逐次对每一个module，chunk进行整理，生成编辑后的代码


简洁版：

1. 读取文件分析模块依赖
2. 对模块进行解析执行(深度遍历)
3. 针对不同的模块使用相应的loader
4. 编译模块，生成抽象语法树AST。
5. 循环遍历AST树，拼接输出js。

<br/>

## 原理-依赖分析和构建

<br/>

<br/>

## 原理-模块打包与异步模块【2】

<br/>

&emsp;&emsp;
通过fs将模块读取成字符串，然后用warp包裹一下，使之成为一个字符串形式的的函数然后调用 vm.runInNewContext 类型的方法，这个字符串会变成一个函数，并传入参数。

```javascript
const str = `require('./moduleA'); const str = require('./moduleB'); console.log(str)`

const functionWarpper = [
    'function(require, module, export){',
    ','
]

const result = functionWarpper[0] + str + functionWarppe[1] // eval new Function()

const vm = require('vm')
vm.runInNewContext
```

&emsp;&emsp;
这些模块的函数会被存放在数组里，然后进行解析执行。module和export都是传入的对象，webpack主要实现require函数，去加载其他模块

```javascript
var moduleDepList = [
  {'./moduleA': 1}, // module[0] 的依赖 他依赖moduleA 且 moduleA的下标在moduleList 中 为 1
  {}
]
function require(id, parentId) {
  var currentModlueId = parentId !== undefined ? moduleDepList[parentId][id] : id
  var module = {exports: {}}
  var moduleFunc = moduleList[currentModlueId]
  moduleFunc(id => require(id, currentModlueId), module, module.exports)
  return module.exports
}
```

&emsp;&emsp;
如果是**异步模块**，则会通过jsonp的形式去加载该模块打包好生成的chunk。 异步加载模块可以使用import和require.ensure函数，函数将会返回一个promise。

```javascript
var cache = {}
require.ensure = function(chunkId, parentId) {
  var currentModlueId = parentId !== undefined ? moduleDepList[parentId][chunkId] : chunkId
  var currentChunk = cache[currentModlueId]

  if (currentChunk === undefined) {
    var $script = document.createElement('script')
    $script.src = `chunk.${chunkId}.js`
    document.body.appendChild($script)

    var promise = new Promise(function(resolve) {
      var chunkCache = [resolve] // 为什么用数组 ，为了保存promise
      chunkCache.status = true // 异步模块加载中 如果有别的包 在 异步加载在模块 那么下面的
      cache[chunkId] = chunkCache
    })
    cache[chunkId].push(promise)
    return promise
  }

  if (currentChunk.status) {
    return currentChunk[1] // promise 这里的就直接返回promise 这样模块只会加载一次
  }
  return currentChunk
}
```

&emsp;&emsp;
上面方法都是公共的，抽离成模板的js文件，webpack负责做依赖分析，并将模块读成函数填充入数组即可。（这里说的只是js的模块）

<br/>

## 原理-热更新

<br/>

1. client 和 server 建立一个 websocket 通信
2. 当有文件发生变动（如fs.watchFile）的时候，webpack编译文件，并通过 websocket 向client发送一条更新消息
3. client 根据收到的hash值，通过ajax获取一个 manifest 描述文件
4. client 根据manifest 获取新的JS模块的代码
5. 当取到新的JS代码之后，会更新 modules tree，（installedModules)调用之前通过 module.hot.accept 注册好的回调，可能是loader提供的，也可能是你自己写的

manifest: 描述资源文件对应关系如，打包后的文件重命名了

```javascript
{
  "a.js": "a.41231243.js"
}
```

<a href="./hotModuleReplace">详细的请点击这里-></a>

<br/>

## 常见plugin

<br/>

- clean-webpack-plugin: 在构建之前删除上一次build的文件夹

- copy-webpack-plugin: 复制文件或文件夹到生成后的目录

- extract-text-webpack | mini-css-extract-plugin: 将所有入口的chunk(entry chunks)中引用的 *.css，移动到独立分离的 CSS 文件

- html-webpack-plugin: 将build后生成的资源以标签的形式嵌入到HTML模板内

- hot-module-replacement: 模块热更新

<br/>

## 常见loader

<br/>

- babel-loader: 语法，源码转换以便能够运行在当前和旧版本的浏览器或其他环境中

- css-loader: 配合style-loader可以解析在js中引入的css文件，并以\<style>便签将css-loader内部样式注入到我们的HTML页面

- file-loader: 可以解析js中require的文件，输出到输出目录并返回 public URL

- html-loader: 可以对HTML模板中指定哪个标签属性组合(tag-attribute combination)元素应该被此 loader 处理

- less-loader: 依赖less，可以将less编译成css

- postcss-loader: 配合一些plugin如cssnano,autoprefixer可以对css进行压缩，优化，自动补足前缀等
  
- scss-loader: 配合node-scss，可以将scss编译成css

- style-loader: 配合css-loader可以解析在js中引入的css文件，并以\<style>便签将css-loader内部样式注入到我们的HTML页面

- url-loader: url-loader 功能类似于 file-loader，但是在文件大小（单位 byte）低于指定的限制时，可以返回一个 DataURL(base64)

<br/>

## 打包优化

## 参考

<br/>

【1】<a href="https://www.webpackjs.com/">webpack官网</a>

【2】<a href="../../../case/engineering/webpack/compile/bundle.template.js">webpack打包</a>

<br/>

## 结束语
部分待补充

如果有疑问或者发现错误，可以在 issues 里提出疑问或勘误。