### 【进阶5-2期】图解原型链及其继承优缺点

**原型链**
---
***

每个对象拥有一个原型对象，通过 `__proto__` 指针指向上一个原型 ，并从中**继承方法和属性**，同时原型对象也可能拥有原型，这样一层一层，最终指向 null，这种关系被称为**原型链(prototype chain)**。

根据规范不建议直接使用 `__proto__`，推荐使用 `Object.getPrototypeOf()`，不过为了行文方便逻辑清晰，下面都以 `__proto__` 代替。

注意上面的说法，原型上的方法和属性被 **继承** 到新对象中，并不是被复制到新对象，我们看下面这个例子。

```javascript
function Foo(name) {
	this.name = name;
}
Foo.prototype.getName = function() {
  	return this.name;
}
Foo.prototype.length = 3;
let foo = new Foo('muyiy'); // 相当于 foo.__proto__ = Foo.prototype
console.dir(foo);
```

原型上的属性和方法定义在 `prototype` 对象上，而非对象实例本身。当访问一个对象的属性 / 方法时，它不仅仅在该对象上查找，还会查找该对象的原型，以及该对象的原型的原型，一层一层向上查找，直到找到一个名字匹配的属性 / 方法或到达原型链的末尾（`null`）。

比如调用 `foo.valueOf()` 会发生什么？

* 首先检查 `foo` 对象是否具有可用的 `valueOf()` 方法。

* 如果没有，则检查 `foo` 对象的原型对象（即 `Foo.prototype`）是否具有可用的 `valueof()` 方法。

* 如果没有，则检查 `Foo.prototype` 所指向的对象的原型对象（即 `Object.prototype`）是否具有可用的 `valueOf()` 方法。这里有这个方法，于是该方法被调用。

**`prototype` 和 `__proto__`**
---

上篇文章介绍了 `prototype` 和 `__proto__` 的区别，其中原型对象 prototype 是构造函数的属性，`__proto__` 是每个实例上都有的属性，这两个并不一样，但 `foo.__proto__` 和 `Foo.prototype` 指向同一个对象。

这次我们再深入一点，原型链的构建是依赖于 `prototype` 还是 `__proto__` 呢？

![](https://camo.githubusercontent.com/420565fca4a18bc803a3c68e767f0dfd7a49b8c1/68747470733a2f2f322e62702e626c6f6773706f742e636f6d2f2d3269694c57367774454f302f55477444582d5a505f6f492f41414141414141414166512f46705346434567316b37512f73313630302f4a6176617363726970742b50726f746f747970616c2b496e6865726974616e63652b4469616772616d2b2d2b6772616e642b706963747572652b2833292e706e67)

`Foo.prototype` 中的 `prototype` 并没有构建成一条原型链，其只是指向原型链中的某一处。原型链的构建依赖于 `__proto__`，如上图通过 `foo.__proto__` 指向 `Foo.prototype`，`foo.__proto__.__proto__` 指向 `Bichon.prototype`，如此一层一层最终链接到 `null`。

```javascript
function Foo() {
  	return 'foo';
}
Foo.prototype.method = function() {
  	return 'method';
}
function Bar() {
  	return 'bar';
}
Bar.prototype = Foo; // Bar.prototype 指向到函数
let bar = new Bar();
console.dir(bar);

bar.method(); // Uncaught TypeError: bar.method is not a function
```

**instanceof 原理及实现**
---

`instanceof` 运算符用来检测 `constructor.prototype` 是否存在于参数 `object` 的原型链上。

```javascript
function C(){} 
function D(){} 

var o = new C();

o instanceof C; // true，因为 Object.getPrototypeOf(o) === C.prototype
o instanceof D; // false，因为 D.prototype 不在 o 的原型链上
```

instanceof 原理就是一层一层查找 `__proto__`，如果和 `constructor.prototype` 相等则返回 true，如果一直没有查找成功则返回 false。

```javascript
instance.[__proto__...] === instance.constructor.prototype
```

知道了原理后我们来实现 instanceof，代码如下。

```javascript
function instance_of(L, R) {//L 表示左表达式，R 表示右表达式
   var O = R.prototype;// 取 R 的显示原型
   L = L.__proto__;// 取 L 的隐式原型
   while (true) { 
       // Object.prototype.__proto__ === null
       if (L === null) 
         return false; 
       if (O === L)// 这里重点：当 O 严格等于 L 时，返回 true 
         return true; 
       L = L.__proto__; 
   } 
}

// 测试
function C(){} 
function D(){} 

var o = new C();

instance_of(o, C); // true
instance_of(o, D); // false
```

**原型链继承**
---
***

原型链继承的本质是**重写原型对象，代之以一个新类型的实例**。如下代码，新原型 `Cat` 不仅有 `new Animal()` **实例**上的全部属性和方法，并且由于指向了 `Animal` 原型，所以还继承了`Animal` **原型**上的属性和方法。

```javascript
function Animal() {
    this.value = 'animal';
}

Animal.prototype.run = function() {
    return this.value + ' is runing';
}

function Cat() {}

// 这里是关键，创建 Animal 的实例，并将该实例赋值给 Cat.prototype
// 相当于 Cat.prototype.__proto__ = Animal.prototype
Cat.prototype = new Animal(); 

var instance = new Cat();
instance.value = 'cat'; // 创建 instance 的自身属性 value
console.log(instance.run()); // cat is runing
```

原型链继承方案有以下缺点：

* 1、多个实例对引用类型的操作会被篡改

* 2、子类型的原型上的 `constructor` 属性被重写了

* 3、给子类型原型添加属性和方法必须在替换原型之后

* 4、创建子类型实例时无法向父类型的构造函数传参

**问题 1 原型属性实例共享**
---

原型链继承方案中，原型实际上会变成另一个类型的实例，如下代码，`Cat.prototype` 变成了 `Animal` 的一个实例，所以 `Animal` 的实例属性 `names` 就变成了 `Cat.prototype` 的属性。

而原型属性上的引用类型值会被所有实例共享，所以多个实例对引用类型的操作会被篡改。如下代码，改变了 `instance1.names` 后影响了 `instance2`。

```javascript
function Animal(){
    this.names = ["cat", "dog"];
}
function Cat(){}

Cat.prototype = new Animal();

var instance1 = new Cat();
instance1.names.push("tiger");
console.log(instance1.names); // ["cat", "dog", "tiger"]

var instance2 = new Cat(); 
console.log(instance2.names); // ["cat", "dog", "tiger"]
```

**问题 2 原型继承时的指向**
---

子类型原型上的 `constructor` 属性被重写了，执行 `Cat.prototype = new Animal()` 后原型被覆盖，`Cat.prototype` 上丢失了 `constructor` 属性， `Cat.prototype` 指向了 `Animal.prototype`，而 `Animal.prototype.constructor` 指向了 `Animal`，所以 `Cat.prototype.constructor` 指向了 `Animal`。

```javascript
Cat.prototype = new Animal(); 
Cat.prototype.constructor === Animal
// true
```

解决办法就是重写 `Cat.prototype.constructor` 属性，指向自己的构造函数 `Cat`。

```javascript
function Animal() {
    this.value = 'animal';
}

Animal.prototype.run = function() {
    return this.value + ' is runing';
}

function Cat() {}
Cat.prototype = new Animal(); 

// 新增，重写 Cat.prototype 的 constructor 属性，指向自己的构造函数 Cat
Cat.prototype.constructor = Cat; 
```

**问题 3 拓展原型的属性和方法**

给子类型原型添加属性和方法必须在替换原型之后，原因在第二点已经解释过了，因为子类型的原型会被覆盖。

```javascript
function Animal() {
    this.value = 'animal';
}

Animal.prototype.run = function() {
    return this.value + ' is runing';
}

function Cat() {}
Cat.prototype = new Animal(); 
Cat.prototype.constructor = Cat; 

// 新增
Cat.prototype.getValue = function() {
  return this.value;
}

var instance = new Cat();
instance.value = 'cat'; 
console.log(instance.getValue()); // cat
```

**属性遮蔽**
---

改造上面的代码，在 `Cat.prototype` 上添加 `run` 方法，但是 `Animal.prototype` 上也有一个 `run` 方法，不过它不会被访问到，这种情况称为属性遮蔽 (property shadowing)。

```javascript
function Animal() {
    this.value = 'animal';
}

Animal.prototype.run = function() {
    return this.value + ' is runing';
}

function Cat() {}
Cat.prototype = new Animal(); 
Cat.prototype.constructor = Cat; 

// 新增
Cat.prototype.run = function() {
  return 'cat cat cat';
}

var instance = new Cat();
instance.value = 'cat'; 
console.log(instance.run()); // cat cat cat
```

那如何访问被遮蔽的属性呢？通过 `__proto__` 调用原型链上的属性即可。

```javascript
// 接上
console.log(instance.__proto__.__proto__.run()); // undefined is runing
```

**其他继承方案**
---
***

原型链继承方案有很多问题，实践中很少会单独使用，日常工作中使用 ES6 Class extends（模拟原型）继承方案即可

**扩展题**
---
***

有以下 3 个判断数组的方法，请分别介绍它们之间的区别和优劣

> Object.prototype.toString.call() 、 instanceof 以及 Array.isArray()

<font size=1>**1. Object.prototype.toString.call()**</font>

每一个继承 Object 的对象都有 `toString` 方法，如果 `toString` 方法没有重写的话，会返回 `[Object type]`，其中 type 为对象的类型。但当除了 Object 类型的对象外，其他类型直接使用 `toString` 方法时，会直接返回都是内容的字符串，所以我们需要使用call或者apply方法来改变toString方法的执行上下文。

```javascript
const an = ['Hello','An'];
an.toString(); // "Hello,An"
Object.prototype.toString.call(an); // "[object Array]"
```

这种方法对于所有基本的数据类型都能进行判断，即使是 null 和 undefined 。

```javascript
Object.prototype.toString.call('An') // "[object String]"
Object.prototype.toString.call(1) // "[object Number]"
Object.prototype.toString.call(Symbol(1)) // "[object Symbol]"
Object.prototype.toString.call(null) // "[object Null]"
Object.prototype.toString.call(undefined) // "[object Undefined]"
Object.prototype.toString.call(function(){}) // "[object Function]"
Object.prototype.toString.call({name: 'An'}) // "[object Object]"
Object.prototype.toString.call([]) // "[object Array]"
Object.prototype.toString.call({}) // "[object Object]"
```

`Object.prototype.toString.call()` 常用于判断浏览器内置对象时。

<font size=1>**2. instanceof**</font>

`instanceof`  的内部机制是通过判断对象的原型链中是不是能找到类型的 `prototype`。

使用 `instanceof`判断一个对象是否为数组，`instanceof` 会判断这个对象的原型链上是否会找到对应的 `Array` 的原型，找到返回 `true`，否则返回 `false`。

```javascript
[]  instanceof Array; // true
```

但 `instanceof` 只能用来判断对象类型，原始类型不可以。并且所有对象类型 instanceof Object 都是 true。

```javascript
[]  instanceof Object; // true
```

<font size=1>**3. Array.isArray()**</font>

* 功能：用来判断对象是否为数组

* instanceof 与 isArray

当检测Array实例时，`Array.isArray` 优于 `instanceof` ，因为 `Array.isArray` 可以检测出 `iframes`

```javascript
var iframe = document.createElement('iframe');
document.body.appendChild(iframe);
xArray = window.frames[window.frames.length-1].Array;
var arr = new xArray(1,2,3); // [1,2,3]

// Correctly checking for Array
Array.isArray(arr);  // true
Object.prototype.toString.call(arr); // true
// Considered harmful, because doesn't work though iframes
arr instanceof Array; // false

[] instanceof Array // true
[] instanceof Object // true
```

* `Array.isArray()` 与 `Object.prototype.toString.call()`

`Array.isArray()`是ES5新增的方法，当不存在 `Array.isArray()` ，可以用 `Object.prototype.toString.call()` 实现。

```javascript
if (!Array.isArray) {
    Array.isArray = function(arg) {
        return Object.prototype.toString.call(arg) === '[object Array]';
    };
}
```
ES6才有的新方法，有兼容性问题
对于性能问题，我测试几遍的结果 Array.isArray 的性能最好，instanceof 比 toString.call 稍微好了一点点

![](https://user-images.githubusercontent.com/15176971/53391076-2c41c780-39d0-11e9-8b9d-2a5b2bc2dc58.png)

<!-- [测试地址](https://jsperf.com/judging-array-type/) -->



**小结**
---
***

* 每个对象拥有一个原型对象，通过 `__proto__` 指针指向上一个原型 ，并从中**继承方法和属性**，同时原型对象也可能拥有原型，这样一层一层，最终指向 `null`，这种关系被称为**原型链 **

* 当访问一个对象的属性 / 方法时，它不仅仅在该对象上查找，还会查找该对象的原型，以及该对象的原型的原型，一层一层向上查找，直到找到一个名字匹配的属性 / 方法或到达原型链的末尾（`null`）。

* 原型链的构建依赖于 `__proto__`，一层一层最终链接到 `null`。

* instanceof 原理就是一层一层查找 `__proto__`，如果和 `constructor.prototype` 相等则返回 true，如果一直没有查找成功则返回 false。

* 原型链继承的本质是**重写原型对象，代之以一个新类型的实例**。