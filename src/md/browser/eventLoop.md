# EventLoop

&emsp;&emsp;
javascript 是一门单线程的脚本语言(这里我们不考虑worker)，或者说只有一个主线程，也就是它一次只能执行一段代码。那么例如像onClick 注册的回调函数、必不可少的ajax等等异步操作需要借助EventLoop来执行。

<br/>

## 执行模型

&emsp;&emsp;
event loop是一个执行模型，在不同的地方有不同的实现。浏览器和NodeJS基于不同的技术实现了各自的Event Loop。
- 浏览器的Event Loop是在html5的规范中明确定义。
- NodeJS的Event Loop则是基于libuv实现的。

<br/>


$\color{green}{浏览器中的EventLoop}$


#### 宏任务和微任务

&emsp;&emsp;
首先来了解下宏任务和微任务，异步任务分为task（宏任务，也可称为macroTask）和microtask（微任务，也叫jobs）两类。
Task:
- setTimeout
- setInterval
- setImmediate (Node独有)
- requestAnimationFrame (浏览器独有)
- I/O
- UI rendering (浏览器独有)
- script标签

MicroTask:
- process.nextTick (Node独有)
- Promise
- MutationObserver

&emsp;&emsp;
当满足执行条件时，task和microtask会被放入各自的队列中，等待放入执行线程执行，我们把这两个队列称为Task Queue(也叫Macrotask Queue)和Microtask Queue。


**基本流程：**

![avatar](/src/img/js/eventLoop_1.jpg)

1. 从任务队列task queue选择中最先进入的任务执行，如果队列为空，则执行microtask queue。
2. 检查该task是否注册了microtask queue，如果有，则按注册顺序依次执行microtask。如果没有则继续下一步。
3. 一些其他操作，例如界面渲染。
4. 重复以上步骤。

这样的一次流程称为event loop的一次循环，即是一次tick。

**例子**
下面我们通过一个例子来具体分析分析：

```Javascript
<script>
  console.log('---------script-1---------')
​
  const box = document.querySelector('.box')
  const observer = new MutationObserver(function (mutationsList, observer) {
    console.log('MutationObserver')
  });
  observer.observe(box, { attributes: true, childList: true, subtree: true });
  box.setAttribute('name', 'rename')
​
  setTimeout(() => {
    console.log('setTimeout--1')
    new Promise(resolve => resolve()).then(() => {
      console.log('in setTimeout promise then')
    })
  })
​
  new Promise((resolve, reject) => {
    setTimeout(() => {
      console.log('in Promise setTimeout')
    })
    console.log('in promise')
    resolve()
  }).then(() => {
    console.log('in promise then')
  })
​
  setTimeout(() => {
    console.log('setTimeout--2')
  })
</script>
<script>
  console.log('---------script-2---------')
</script>
​
```

<br/>

---

<div align="center"><font color=blue size=4>0 1</font></div>

首先执行第一个script：   
1. console.log同步代码直接输出
2. 注册了一个MutationObserver，添加到当前task的microtask queue
3. setTimeout定时器，往task queue中添加了一个宏任务
4. new Promise，首先执行里面的同步代码，添加定时器和console.log，再在当前task的microtask queue中添加一个promise的microtask
5. setTimeout定时器，在往task queue中添加了一个宏任务

<br/>
<div align="center"><font color=blue size=4>0 2</font></div>
<br/>

第一个task中的同步任务执行完成，查看当前task的microtask queue，不为空，则依次执行队列中的microtask。
1. 执行MutationObserver的回调函数
2. 执行Promise.then的回调函数

<br/>
<div align="center"><font color=blue size=4>0 3</font></div>
<br/>

microtask queue执行完毕，script1的task从执行栈弹出，script2压入栈执行。输出console.log。script2 中没有注册异步任务，循环后执行下一个task - setTimeout1。
1. 输出同步代码console.log
2. 往自身的microtask queue中添加一个promise的microtask

<br/>
<div align="center"><font color=blue size=4>0 4</font></div>
<br/>

接着
1. 执行自身的microtask queue，promise.then注册的函数，控制台输出in setTimeout promise then。
2. 执行完其他操作后，进入下一轮循环，执行在promise中注册的定时器，该task中没有microtask，继续下一轮循环，执行setTimemout2。
3. 执行完后，js线程空闲等他其他操作。

&emsp;&emsp;
浏览器下的执行结果和我们分析的一致

![avatar](/src/img/js/eventLoop_2.jpg)


<br/>

$\color{green}{Node中的EventLoop}$

&emsp;&emsp;
node的event Loop是基于libuv 库实现的。我们看看libuv中的主要函数uv_run（只保留mode=UV_RUN_DEFAULT 默认轮询模式）。
```C
int uv_run(uv_loop_t* loop, uv_run_mode mode) {
  int timeout;
  int r;
  int ran_pending;
​
​  r = uv__loop_alive(loop);
  if (!r)
    uv__update_time(loop);
​
  while (r != 0 && loop->stop_flag == 0) {
    uv__update_time(loop);
    uv__run_timers(loop); // 这个阶段执行timer（setTimeout、setInterval）的回调
​
    ran_pending = uv__run_pending(loop);
    uv__run_idle(loop);
    uv__run_prepare(loop);
​
    timeout = 0;
    if (mode == UV_RUN_DEFAULT)
      timeout = uv_backend_timeout(loop);
​
    uv__io_poll(loop, timeout); // 获取新的I/O事件, 适当的条件下node将阻塞在这里
    uv__run_check(loop); // 执行 setImmediate() 的回调
    uv__run_closing_handles(loop); // 执行close事件的callback，例如socket.on("close",func)
​
    r = uv__loop_alive(loop);
    if (mode == UV_RUN_ONCE || mode == UV_RUN_NOWAIT)
      break;
  }
​
  /* The if statement lets gcc compile it to a conditional store. Avoids
   * dirtying a cache line.
   */
  if (loop->stop_flag != 0)
    loop->stop_flag = 0;
​
  return r;
}
```


while循环中就是我们的eventLoop。用文字描述描述eventLoop的步骤如下：

1. timers：执行满足条件的setTimeout、setInterval回调。
2. I/O callbacks：是否有已完成的I/O操作的回调函数，来自上一轮的poll残留。
3. idle，prepare：可忽略
4. poll：等待还没完成的I/O事件，会因timers和超时时间等结束等待。
5. check：执行setImmediate的回调。
6. close callbacks：关闭所有的closing handles，一些onclose事件。
7. 重复以上步骤。

```
   ┌───────────────────────┐
┌─>│        timers         │<————— 执行 setTimeout()、setInterval() 的回调
│  └──────────┬────────────┘
|             |<-- 执行所有 Next Tick Queue 以及 MicroTask Queue 的回调
│  ┌──────────┴────────────┐
│  │     pending callbacks │<————— 执行由上一个 Tick 延迟下来的 I/O 回调（待完善，可忽略）
│  └──────────┬────────────┘
|             |<-- 执行所有 Next Tick Queue 以及 MicroTask Queue 的回调
│  ┌──────────┴────────────┐
│  │     idle, prepare     │<————— 内部调用（可忽略）
│  └──────────┬────────────┘     
|             |<-- 执行所有 Next Tick Queue 以及 MicroTask Queue 的回调
|             |                   ┌───────────────┐
│  ┌──────────┴────────────┐      │   incoming:   │ - (执行几乎所有的回调，除了 close callbacks 以及 timers 调度的回调和 setImmediate() 调度的回调，在恰当的时机将会阻塞在此阶段)
│  │         poll          │<─────┤  connections, │ 
│  └──────────┬────────────┘      │   data, etc.  │ 
│             |                   |               | 
|             |                   └───────────────┘
|             |<-- 执行所有 Next Tick Queue 以及 MicroTask Queue 的回调
|  ┌──────────┴────────────┐      
│  │        check          │<————— setImmediate() 的回调将会在这个阶段执行
│  └──────────┬────────────┘
|             |<-- 执行所有 Next Tick Queue 以及 MicroTask Queue 的回调
│  ┌──────────┴────────────┐
└──┤    close callbacks    │<————— socket.on('close', ...)
   └───────────────────────┘
```


<br/>

$\color{green}{经典例子}$

最后给大家讲一个经典例子

```
setTimeout(()=>{
  console.log('timer1')
  Promise.resolve().then(function() {
    console.log('promise1')
  })
}, 0) 
setTimeout(()=>{
  console.log('timer2')
  Promise.resolve().then(function() {
    console.log('promise2')
  })
}, 0)
```

结果如下：

- 浏览器环境：time1，promise1，time2，promise2
- node11以下：time1，time2，promise1，promise2
- node11及以上：time1，promise1，time2，promise2

在 node 11 版本中，node 下 Event Loop 已经与浏览器趋于相同。我们可以用浏览器的微任务和宏任务解释，11版本前的timer，由于到期时间相近，会在timer阶段合并执行。所以打出time1后，打印time2。

<br/>

### 结束语
<a href="../../case/html/js/eventLoop.html" >eventLoop - html代码文件</a>

<a href="../../case/node/eventLoop.js" >eventLoop - nodejs</a>

如果有错误或者不严谨的地方，欢迎指正~