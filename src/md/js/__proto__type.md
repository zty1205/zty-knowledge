# 原型和原型链

<br/>

## __proto__ 和 prototype

<br/>

$\color{green}{普通的构造函数和对象}$

&emsp;&emsp;
首先熟记一下三点：

- \__proto__ 是对象实例才有的属性，指向对象的原型。
- prototype 是构造函数才有的属性，该属性指向了一个对象，这个对象正是调用该构造函数而创建的实例的原型
- 实例的__proto__属性 和 构造函数的 prototype 都指向该对象原型-

&emsp;&emsp;
那么什么是原型呢？每一个JavaScript对象(null除外)在创建的时候就会关联另一个对象，这个对象就是我们所说的原型，每一个对象都会从原型"继承"属性和方法。

&emsp;&emsp;
我们从一个例子来看看，我们先声明一个构造函数 Person:

```Javascript
function Person(name) { 
  this.name = name
}
```

&emsp;&emsp;
这里构造函数Person的 prototype将指向他们的对象原型，对象原型的构造函数将指向 Person 即：
![avatar](/src/img/js/proto_1.png)

&emsp;&emsp;
那么实例又会有什么样的联系呢？我们new一个实例p, new一个对象的原理如下：
1. 创建一个空对象，构造函数中的this指向这个空对象
2. 这个新对象的__proto__设置为构造函数的prototype
3. 执行构造函数方法，属性和方法被添加到this引用的对象中
4. 如果构造函数中没有返回其它对象，那么返回this，即创建的这个的新对象，否则，返回构造函数中返回的对象。

&emsp;&emsp;
从new一个对象的原理上我们可以看到实例的 __proto__ 也指向了原型实例。添加后的图如下。
![avatar](/src/img/js/proto_2.png)

&emsp;&emsp;
构造函数，实例，原型实例呈现一个三角循环。但是原型实例，和构造函数并不能访问实例，因为你可以同时new无数个实例，但构造函数和原型实例都只有一个。

&emsp;&emsp;
继续往上查看，Person.prototype的__proto__将指向Object.prototype，即Object构造函数的prototype。

```Javascript
Person.prototype.__proto__ === Object.prototype
```

&emsp;&emsp;
在继续往上Object.prototype.\__proto__ 将等于null。到达Object.prototype时已经到了顶级的原型。
![avatar](/src/img/js/proto_3.png)

<br/>

$\color{green}{Function 和 Object}$

&emsp;&emsp;
下面我们看看Function和Object。直接给出结论：
![avatar](/src/img/js/proto_4.png)

&emsp;&emsp;
我们归纳以下几点：

- Function的prototype和__proto__属性都指向f () 匿名函数
- Object作为构造函数时，他的prototype指向Object.prototype对象原型，作为实例时，他的__proto__指向匿名函数。我们可以认为Function实例和Object实例都是继承于该匿名函数。
- 匿名函数作为$\color{red}{顶级构造函数}$，他不需要prototype属性，即prototype=undefined，当作为对象时，他的对象原型是Object.prototype。
- Object.prototype作为$\color{red}{顶级构造对象}$，他的__proto__等于null，表示继承于一个空的对象。没有prototype属性。
  
<br/>

## 原型链

&emsp;&emsp;
在上面我们通过Person的示意图中看到的，用 __proto__ 链接的这条就是我们的原型链。原型链用于查找对象上的属性，当属性未从当前的对象上获取到的时候会从该原型链上查找，直到查到相应的属性。
![avatar](/src/img/js/proto_5.png)

&emsp;&emsp;
我们用一个具体的例子说明：

```Javascript
function Person(name) { 
  this.name = name
}
let p = new Person('zty')
let p1 = Object.create(p)
p1.name = "rename"
```

&emsp;&emsp;
Object.create函数会以对象p为原型对象创建对象P1，当我们访问p1.name时，会得到‘rename’。如果我们将该属性删除，那么访问p1.name会得到什么呢？让我们大胆猜想，基于原型链的特性，在本身未找到name属性，它会沿着原型链向上查找name属性。那么答案就应该是name="zty"。
![avatar](/src/img/js/proto_6.jpg)
&emsp;&emsp;
从控制台中验证答案。结果和猜想一致

<br/>

## 几个和原型相关的函数

- Object.create
- Object.setPrototypeOf 和 Object.getPrototypeOf
- instanceof 和 isPrototypeOf


&emsp;&emsp;
Object.create(obj) 可以基于某个对象为原型创建对象。

&emsp;&emsp;
__proto__是一个内部属性,不是一个正式的对外的API。我们访问obj.__proto__其实使用的就是Object.getPrototypeOf(obj)。Object.setPrototypeOf则可以为对象附加原型链。如同上面使用的Object.create，下面两种的效果是一致的。

```javascript
let p1 = Object.create(p)
// 等价于
let p1 = {}
Object.setPrototypeOf(p1, p)
```

&emsp;&emsp;
instanceof 运算符用于检测构造函数的 prototype 属性是否出现在某个实例对象的原型链上。

&emsp;&emsp;
isPrototypeOf 则用于判断一个对象是否被包含在另一个对象的原型链中

```javascript
p1 instanceof Person // true
// p1继承于p 而p.__proto === Person.prototype
​
p.isPrototypeOf(p1) // true
// p1继承p
```

## 结束语
<a href="../../case/html/js/__proto__type.html" >原型和原型链 - html代码文件</a>

如果有错误或者不严谨的地方，欢迎指正~