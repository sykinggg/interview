### 【进阶2-3期】JavaScript深入之闭包面试题解

作用域指的是一个变量和函数的作用范围，**JS中函数内声明的所有变量在函数体内始终是可见的**，在ES6前有全局作用域和局部作用域，但是没有块级作用域（catch只在其内部生效），局部变量的优先级高于全局变量。

**作用域**
---

<font size=1>**变量提升**</font>

```javascript
var scope="global";
function scopeTest(){
    console.log(scope);
    var scope="local"  
}
scopeTest(); //undefined
```

上面的代码输出是`undefined`，这是因为局部变量`scope`变量提升了，等效于下面

```javascript
var scope="global";
function scopeTest(){
    var scope;
    console.log(scope);
    scope="local"  
}
scopeTest(); //undefined
```

**注意**，如果在局部作用域中忘记var，那么变量就被声明为全局变量。

<font size=1>**没有块级作用域**</font>

```javascript
var data = [];

for (var i = 0; i < 3; i++) {
    data[i] = function () {
        console.log(i);
    };
}

data[0]();	// 3
data[1]();	// 3
data[2]();	// 3
```

<font size=1>**作用域链**</font>

每个函数都有自己的执行上下文环境，当代码在这个环境中执行时，会创建变量对象的作用域链，作用域链是一个对象列表或对象链，它保证了变量对象的有序访问。

作用域链的开始是当前代码执行环境的变量对象，常被称之为“活跃对象”（AO），变量的查找会从第一个链的对象开始，如果对象中包含变量属性，那么就停止查找，如果没有就会继续向上级作用域链查找，直到找到全局对象中

<font size=1>**闭包**</font>

```javascript
function createClosure(){
    var name = "jack";
    return {
        setStr:function(){
            name = "rose";
        },
        getStr:function(){
            return name + ":hello";
        }
    }
}
var builder = new createClosure();
builder.setStr();
console.log(builder.getStr()); //rose:hello
```

上面在函数中返回了两个闭包，这两个闭包都维持着对外部作用域的引用。闭包中会将外部函数的自由对象添加到自己的作用域链中，所以可以通过内部函数访问外部函数的属性，这也是javascript模拟私有变量的一种方式。

**闭包面试题解**
---

由于作用域链机制的影响，闭包只能取得内部函数的最后一个值，这引起的一个副作用就是如果内部函数在一个循环中，那么变量的值始终为最后一个值。

```javascript
var data = [];

for (var i = 0; i < 3; i++) {
    data[i] = function () {
        console.log(i);
    };
}

data[0]();	// 3
data[1]();	// 3
data[2]();	// 3
```

<font size=1>**方法1：立即执行函数**</font>

```javascript
for (var i = 0; i < 3; i++) {
    (function(num) {
        setTimeout(function() {
            console.log(num);
        }, 1000);
    })(i);
}
// 0
// 1
// 2
```

<font size=1>**方法2：返回一个匿名函数赋值**</font>

```javascript
var data = [];

for (var i = 0; i < 3; i++) {
    data[i] = (function (num) {
        return function(){
            console.log(num);
        }
    })(i);
}

data[0]();	// 0
data[1]();	// 1
data[2]();	// 2
```

无论是**立即执行函数**还是**返回一个匿名函数赋值**，原理上都是因为变量的按值传递，所以会将变量`i`的值复制给实参`num`，在匿名函数的内部又创建了一个用于访问`num`的匿名函数，这样每个函数都有了一个`num`的副本，互不影响了。

<font size=1>**方法3：使用ES6中的let**</font>

```javascript
var data = [];

for (let i = 0; i < 3; i++) {
    data[i] = function () {
        console.log(i);
    };
}

data[0]();
data[1]();
data[2]();
```

解释下**原理：**

```javascript
var data = [];// 创建一个数组data;

// 进入第一次循环
{ 
	let i = 0; // 注意：因为使用let使得for循环为块级作用域
	           // 此次 let i = 0 在这个块级作用域中，而不是在全局环境中
    data[0] = function() {
    	console.log(i);
	};
}
```

循环时，`let`声明`i`,所以整个块是块级作用域，那么`data[0]`这个函数就成了一个闭包。这里用｛｝表达并不符合语法，只是希望通过它来说明let存在时，**这个for循环块是块级作用域，而不是全局作用域**。

上面的块级作用域，就像函数作用域一样，函数执行完毕，其中的变量会被销毁，但是因为这个代码块中存在一个闭包，闭包的作用域链中引用着块级作用域，**所以在闭包被调用之前，这个块级作用域内部的变量不会被销毁**。

```javascript
// 进入第二次循环
{ 
	let i = 1; // 因为 let i = 1 和上面的 let i = 0     
	           // 在不同的作用域中，所以不会相互影响
	data[1] = function(){
         console.log(i);
	}; 
}
```

当执行`data[1]()`时，进入下面的执行环境。

```javascript
{ 
    let i = 1; 
    data[1] = function(){
        console.log(i);
    }; 
}
```

在上面这个执行环境中，它会首先寻找该执行环境中是否存在`i`，没有找到，就沿着作用域链继续向上到了其所在的块作用域执行环境，找到了`i = 1`,于是输出了`1`。

**思考题**
---

代码1：

```javascript
var scope = "global scope";
function checkscope(){
    var scope = "local scope";
    function f(){
        return scope;
    }
    return f;
}

checkscope()();  
```

代码2：

```javascript
var scope = "global scope";
function checkscope(){
    var scope = "local scope";
    function f(){
        return scope;
    }
    return f;
}

var foo = checkscope(); 
foo();
```

上面的两个代码中，`checkscope()`执行完成后，闭包f所引用的自由变量`scope`会被垃圾回收吗？为什么？