### 【进阶1-2期】JavaScript深入之执行上下文栈和变量对象

JS是单线程的语言，执行顺序肯定是顺序执行，但是JS 引擎并不是一行一行地分析和执行程序，而是一段一段地分析执行，会先进行编译阶段然后才是执行阶段。

例子一：**变量提升**

```javascript
foo;  // undefined
var foo = function () {
    console.log('foo1');
}

foo();  // foo1，foo赋值

var foo = function () {
    console.log('foo2');
}

foo(); // foo2，foo重新赋值
```

例子二：**函数提升**

```javascript
foo();  // foo2
function foo() {
    console.log('foo1');
}

foo();  // foo2

function foo() {
    console.log('foo2');
}

foo(); // foo2
```

例子三：声明优先级，**函数 > 变量**

```javascript
foo();  // foo2
var foo = function() {
    console.log('foo1');
}

foo();  // foo1，foo重新赋值

function foo() {
    console.log('foo2');
}

foo(); // foo1
```

上面三个例子中，第一个例子是变量提升，第二个例子是函数提升，第三个例子是函数声明优先级高于变量声明。

**需要注意**的是同一作用域下存在多个同名函数声明，后面的会替换前面的函数声明。

**执行上下文**

执行上下文总共有三种类型

* **全局执行上下文：**只有一个，浏览器中的全局对象就是 window 对象，this 指向这个全局对象。
* **函数执行上下文：**存在无数个，只有在函数被调用的时候才会被创建，每次调用函数都会创建一个新的执行上下文。
* **Eval 函数执行上下文：** 指的是运行在 eval 函数中的代码，很少用而且不建议使用。

**执行上下文栈**

因为JS引擎创建了很多的执行上下文，所以JS引擎创建了执行上下文**栈**（Execution context stack，ECS）来**管理**执行上下文。

当 JavaScript 初始化的时候会向执行上下文栈压入一个全局执行上下文，我们用 **globalContext** 表示它，并且只有当整个应用程序结束的时候，执行栈才会被清空，所以程序结束之前， 执行栈最底部永远有个 globalContext。

```javascript
ECStack = [		// 使用数组模拟栈
    globalContext
];
```

具体执行过程如下图所示

![执行流程](https://camo.githubusercontent.com/2b271448ad38e8fde43f28db066af7dbe356cbb3/68747470733a2f2f757365722d676f6c642d63646e2e786974752e696f2f323031382f31312f352f313636653235386531643032383161363f696d61676556696577322f302f772f313238302f682f3936302f666f726d61742f776562702f69676e6f72652d6572726f722f31)

**找不同**

有如下两段代码，执行的结果是一样的，但是两段代码究竟有什么不同？

```javascript
var scope = "global scope";
function checkscope(){
    var scope = "local scope";
    function f(){
        return scope;
    }
    return f();
}
checkscope();
```

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

答案是 执行上下文栈的变化不一样。

第一段代码：