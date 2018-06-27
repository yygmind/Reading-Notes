# 你不知道的JavaScript 上卷

## 第一部分 作用域和闭包

### 第1章 作用域是什么

* 问题1：变量储存在哪里？
* 问题2：程序需要时如何找到它们？



#### 1.1 编译原理

JavaScript语言是“动态”或“解释执行”语言，但事实上是一门编译语言。但它不是提前编译的，编译结果也不能在分布式系统中移植。

传统编译语言流程中，程序在执行之前会经历三个步骤，统称为“编译”。

* 分词/词法分析（Tokenizing/Lexing）

  将由字符组成的字符串分解成（对编程语言来说）有意义的代码块。

  ```Js
  var a = 2;
  ```

  上面这段程序会被分解成以下词法单元：var、a、=、2、;。

  空格是否会被当做词法单元，取决于空格在这门语言中是否有意义。

  

* 解析/语法分析（Parsing）

  将词法单元流（数组）转换成一个由元素逐级嵌套所组成的代表了程序语法结构的数。这个数被称作`抽象语法树`（Abstract Syntax Tree, AST）。

  ```Js
  var a = 2;
  ```

  以上代码的抽象语法树如下所示：

  * VariableDeclaration 顶级节点
    * Identifier 子节点，值为a
    * AssignmentExpression 子节点
      * NumericLiteral 子节点，字为2

  

* 代码生成

  将`AST`转换成可执行代码的过程。过程与语言、目标平台等相关。

  简单来说就是可以通过某种方法将`var a = 2;`的AST转化为一组机器指令。用来创建一个叫做a的变量（包括分配内存等），并将一个值存储在a中。

  

#### 1.2 理解作用域

##### 1.2.1 演员表

* 引擎：从头到尾负责整个JavaScript程序的编译和执行。
* 编译器：负责语法分析和代码生成等
* 作用域：负责收集并维护由所有声明的标识符（变量、函数）组成的一系列查询，并实施一套非常严格的规则，确定当前执行的代码对这些标识符的访问权限。

##### 1.2.2 对话

`var a = 2;`存在2个不同的声明。

* 1、编译器在编译时处理（`var a`）：在当前作用域中声明一个变量（如果之前没有声明过）。

  ```Flow
  st=>start: Start
  e=>end: End
  op1=>operation: 分解成词法单元
  op2=>operation: 解析成树结构AST
  cond=>condition: 当前作用域存在变量a?
  op3=>operation: 忽略此声明，继续编译
  op4=>operation: 在当前作用域集合中声明新变量a
  op5=>operation: 生成代码
  st->op1->op2->cond
  cond(yes)->op3->op5->e
  cond(no)->op4->op5->e
  ```

* 2、引擎在运行时处理（`a = 2`）：在作用域中查找该变量，如果找到就对变量赋值。

```Flow
st=>start: Start
e=>end: End
cond=>condition: 当前作用域存在变量a?
cond2=>condition: 全局作用域?
op1=>operation: 引擎使用这个变量a
op2=>operation: 引擎向上一级作用域查找变量a
op3=>operation: 引擎把2赋值给变量a
op4=>operation: 举手示意，抛出异常
st->cond
cond(yes)->op1->op3->e
cond(no)->cond2(no)->op2(right)->cond
cond2(yes)->op4->e
```

##### 1.2.3 LHS和RHS查询 

`L`和`R`分别代表一个赋值操作的左侧和右侧，当变量出现在赋值操作的左侧时进行`LHS`查询，出现在赋值操作的**`非左侧`**时进行`RHS`查询。

* LHS查询（左侧）：找到变量的容器本身，然后对其赋值

* RHS查询（非左侧）：查找某个变量的值，可以理解为 `retrieve his source value`，即取到它的源值

```Js
function foo(a) {
    console.log( a ); // 2
}

foo(2);
```

上述代码共有1处LHS查询，3处RHS查询。

* LHS查询有：
  * 隐式的`a = 2`中，在`2`被当做参数传递给`foo(…)`函数时，需要对参数`a`进行LHS查询

* RHS查询有：

  * 最后一行`foo(...)`函数的调用需要对foo进行RHS查询

  * `console.log( a );`中对`a`进行RHS查询

  * `console.log(...)`本身对`console`对象进行RHS查询

    

#### 1.3 作用域嵌套

遍历嵌套作用域链的规则：引擎从当前的执行作用域开始查找变量，如果找不到就向上一级继续查找。当抵达最外层的全局作用域时，无论找到还是没有找到，查找过程都会停止。



#### 1.4 异常

`ReferenceError`和作用域判别失败相关，`TypeError`表示作用域判别成功了，但是对结果的操作是非法或不合理的。

* RHS查询在作用域链中搜索不到所需的变量，引擎会抛出`ReferenceError`异常。
* 非严格模式下，LHS查询在作用域链中搜索不到所需的变量，全局作用域中会创建一个具有该名称的变量并返还给引擎。
* 严格模式下（ES5开始，禁止自动或隐式地创建全局变量），LHS查询失败会抛出`ReferenceError`异常
* 在RHS查询成功情况下，对变量进行不合理的操作，引擎会抛出`TypeError`异常。（比如对非函数类型的值进行函数调用，或者引用null或undefined类型的值中的属性）



#### 1.5 小结

`var a = 2`被分解成2个独立的步骤。

* 1、`var a`在其作用域中声明新变量
* 2、`a = 2`会LHS查询a，然后对其进行赋值

#####  

### 第2章 词法作用域

#### 2.1 词法阶段

词法作用域是定义在词法阶段的作用域，是由写代码时将变量和块作用域写在哪里来决定的，所以在词法分析器处理代码时会保持作用域不变。（不考虑欺骗词法作用域情况下）

##### 2.1.1 查找

* 作用域查找会在找到第一个匹配的标识符时停止。

* 遮蔽效应：在多层嵌套作用域中可以定义同名的标识符，内部的标识符会“遮蔽”外部的标识符。

* 全局变量会自动变成全局对象的属性，可以间接的通过对全局对象属性的引用来访问。通过这种技术可以访问那些被同名变量所遮蔽的全局变量，但是非全局的变量如果被遮蔽了，无论如何都无法被访问到。

  ```js
  window.a
  ```

* 词法作用域只由函数被声明时所处的位置决定。

* 词法作用域查找只会查找一级标识符，比如a、b、c。对于`foo.bar.baz`，词法作用域只会查找`foo`标识符,找到之后，**对象属性访问规则**会分别接管对`bar`和`baz`属性的访问。



#### 2.2 欺骗词法

欺骗词法作用域会导致性能下降。以下两种方法**不推荐使用**

##### 2.2.1 eval

`eval(..)`函数可以接受一个字符串为参数，并将其中的内容视为好像在书写时就存在于程序中这个位置的代码。

```Js
function foo (str, a) {
    eval( str ); // 欺骗！
    console.log( a, b );
}

var b = 2;
foo( "var b = 3;", 1 ); // 1, 3
```

`eval('var b = 3')`会被当做本来就在那里一样来处理。

* 非严格模式下，如果`eval(..)`中所执行的代码包含一个或多个声明，会在运行期修改书写期的词法作用域。上述代码中在`foo(..)`内部创建了一个变量b，并遮蔽了外部作用域中的同名变量。
* 严格模式下，`eval(..)`在运行时有自己的词法作用域，其中的声明无法修改作用域。

```Js
function foo (str) {
    "use strict"; 
    eval( str ); 
    console.log( a ); // ReferenceError: a is not defined
}

foo( "var a = 2;" ); 
```

* `setTimeout(..)`和`setInterval(..)`的第一个参数可以是字符串，会被解释为一段动态生成的函数代码。**已过时，不要使用**
* `new Function(..) `的最后一个参数可以接受代码字符串（前面的参数是新生成的函数的形参）。**避免使用**



##### 2.2.2 with

`with`通常被当做重复引用同一个对象中的多个属性的快捷方式，可以**不需要重复引用对象本身**。

```Js
var obj = {
    a: 1,
    b: 2,
    c: 3
};

// 单调乏味的重复“obj”
obj.a = 2;
obj.b = 3;
obj.c = 4;

// 简单的快捷方式
with (obj) {
	a = 3;
    b = 4;
    c = 5;
}
```

with可以将一个没有或有多个属性的对象处理为一个完全隔离的词法作用域，这个对象的属性会被处理为定义在这个作用域中的词法标识符。

这个块内部正常的var声明并不会被限制在这个块的作用域中，而是被添加到with所处的函数作用域中。

```Js
function foo(obj) {
    with (obj) {
        a = 2;
    }
}

var o1 = {
    a: 3
};

var o2 = {
    b : 3
}

foo( o1 );
console.log( o1.a ); // 2

foo( o2 );
console.log( o2.a ); // undefined
console.log( a ); // 2 -- 不好，a被泄露到全局作用域上了！
```

上面例子中，创建了`o1`和`o2`两个对象。其中一个有`a`属性，另一个没有。在`with(obj){..}`内部是一个LHS引用，并将2赋值给它。

* `o1`传递进去后，with声明的作用域是`o1`,`a = 2`赋值操作找到`o1.a`并将2赋值给它。
* `o2`传递进去后，作用域`o2`中并没有`a`属性，因此进行正常的LHS标识符查找，o2的作用域、`foo(..)`的作用域和全局作用域都没有找到标识符a，因此当`a = 2`执行时，自动创建了一个全局变量（非严格模式），所以`o2.a`保持undefined。



##### 2.2.3 性能

* JavaScript引擎会在编译阶段进行数项的性能优化，其中有些优化依赖于能够根据代码的词法进行静态分析，并预先确定所有变量和函数的定义位置，才能在执行过程中快速找到标识符。
* 引擎在代码中发现`eval(..)`或`with`，它只能简单的**假设**关于标识符位置的判断都是无效的。因为无法在词法分析阶段明确知道`eval(..)`会接收到什么代码，这些代码会如何对作用域进行修改，也无法知道传递给`with`用来创建词法作用域的对象的内容到底是什么。
* 悲观情况下如果出现了`eval(..)`或with，所有的优化**可能**都是无意义的，最简单的做法就是**完全不做**任何优化。代码运行起来一定会变得非常慢。



#### 2.3 小结

词法作用域意味着作用域是由书写代码时函数声明的位置来决定的。

编译的词法分析阶段基本能够知道全部标识符在哪里以及是如何声明的，从而能够预测在执行过程中如何对它们进行查找。

有以下两个机制可以“欺骗”词法作用域：

* `eval(..)`：对一段包含一个或多个声明的”代码“字符串进行演算，借此来修改已经存在的词法作用域（**运行时**）。
* `with`：将一个对象的引用**当做**作用域来处理，将对象的属性当做作用域中的标识符来处理，创建一个新的词法作用域（**运行时**）。

副作用是引擎无法在编译时对作用域查找进行优化。因为引擎只能谨慎地认为这样的优化是无效的，使用任何一个都将导致代码运行变慢。**不要使用它们**



### 第3章 函数作用域和块作用域

#### 3.1 函数中的作用域

属于这个函数的全部变量都可以在整个函数的范围内使用及复用（事实上在嵌套的作用域中也可以使用）。

```Js
function foo(a) {
    var b = 2;
    
    // 一些代码
    
    function bar() {
        // ...
    }
    
    // 更多的代码
    
    var c = 3;
}
```

`foo(..)`作用域中包含了标识符（变量、函数）a、b、c和bar。无论标识符声明出现在作用域中的何处，这个标识符所代表的变量或函数都将附属于所处的作用域。

全局作用域只包含一个标识符：`foo`。



#### 3.2 隐藏内部实现

最小特权原则（最小授权或最小暴露原则）：在软件设计中，应该最小限度地暴露必要内容，而将其他内容都”隐藏“起来，比如某个模块或对象的API设计。

```Js
function doSomething(a) {
    function doSomethingElse(a) {
        return a - 1;
    }
    
    var b;
    
    b = a + doSomethingElse( a * 2 );
    
    console.log( b * 3 );
}

doSomething( 2 ); // 15
```

`b`和`doSomethingElse(..)`都无法从外部被访问，而只能被`doSomething(..)`所控制，设计上将具体内容私有化了。



##### 3.2.1 规避冲突

”隐藏“作用域中的变量和函数带来的另一个好处是可以避免同名标识符之间的冲突。

```Js
function foo() {
    function bar(a) {
        i = 3; // 修改for循环所属作用域中的i
        console.log( a + i );
    }
    
    for (var i = 0; i < 10; i++) {
        bar( i * 2 ); // 糟糕，无限循环了！
    }
}
foo();
```

`bar(..)`内部的赋值表达式`i = 3`意外的覆盖了声明在`foo(..)`内部for循环中的i。



> 解决方案：

* 声明一个本地变量，任何名字都可以，例如`var i = 3`。
* 采用一个完全不同的标识符名称，例如`var j = 3`。



规避变量冲突的典型例子：

* 全局命名空间

  第三方库会在全局作用域中声明一个名字足够独特的变量，通常是一个对象，这个对象被用作库的命名空间，所有需要暴露给外界的功能都会成为这个对象（命名空间）的属性，而不是将自己的标识符暴露在顶级的词法作用域中。

* 模块管理

  任何库无需将标识符加入到全局作用域中，而是通过依赖管理器的机制将库的标识符显示的导入到另外一个特定的作用域中。



#### 3.3 函数作用域 

```Js
var a = 2;

function foo() { // <-- 添加这一行
    
    var a = 3;
    console.log( a ); // 3
    
} // <-- 以及这一行
foo(); // <-- 以及这一行

console.log( a ); // 2
```

上述函数作用域虽然可以将内部的变量和函数定义”隐藏“起来，但是会导致以下2个额外问题。

* 必须声明一个具名函数`foo()`，意味着`foo`这个名称本身”污染“了所在的作用域。
* 必须显示地通过函数名`foo()`调用这个函数才能运行其中的代码。



> 解决方案：

```Js
var a = 2;

(function foo(){ // <-- 添加这一行
    
    var a = 3;
    console.log( a ); // 3
    
})(); // <-- 以及这一行

console.log( a ); // 2
```

上述代码包装函数的声明以`(function...`开始，函数会被当做函数表达式而不是一个标准的函数声明来处理。

* 区分**函数声明**和**函数表达式**最简单的方法是看function关键字出现在声明中的位置（不仅仅是一行代码，而是整个声明中的位置）。
  * 函数声明：`function`是声明中的第一个词
  * 函数表达式：**不是**声明中的第一个词
* **函数声明**和**函数表达式**之间最重要的区别是它们的名称标识符将会绑定在何处。
  * 第一个片段中，`foo`被绑定在所在作用域中，可以直接通过`foo()`来调用它。
  * 第二个片段中，`foo`被绑定在函数表达式自身的函数中，而不是所在的作用域。`(function foo(){ .. }`中`foo`只能在`..`所代表的位置中被访问，外部作用域不行。`foo`变量名被隐藏在自身中意味着不会非必要地污染外部作用域。



##### 3.3.1 匿名和具名

```Js
setTimeout( function() {
    console.log("I wait 1 second!");
}, 1000 );
```

上述是**匿名函数表达式**，因为`function()..`没有名称标识符。

函数表达式可以匿名，但函数声明不可以省略函数名。

匿名函数表达式有以下缺点：

* 在栈追踪中不会显示出有意义的函数名，会使得调试困难。
* 没有函数名，当函数需要引用自身时只能使用已经**过期**的`arguments.callee`引用
  * 递归
  * 事件触发后事件监听器需要解绑自身
* 匿名函数省略了对于代码可读性/可理解性很重要的函数名。



> 解决方案：

**行内函数表达式**可以解决上述问题，始终给函数表达式命名是一个最佳实践。

```Js
setTimeout( function timeoutHandler() { // <-- 快看，我有名字了！
    console.log( "I waited 1 second!" );
}, 1000 );
```



##### 3.3.2 立即执行函数表达式

立即执行函数表达式（IIFE，Immediately Invoked Function Expression）

* 匿名/具名函数表达式

  第一个（ ）将函数变成表达式，第二个（ ）执行了这个函数

  ```Js
  var a = 2;
  (function IIFE() {
      
      var a = 3;
      console.log( a ); // 3
      
  })();
  
  console.log( a ); // 2
  ```

  

*  改进型`(function(){ .. }())`

  用来调用的（ ）被移进了用来包装的（ ）中。



* 当做函数调用并传递参数进去

  ```Js
  var a = 2;
  (function IIFE( global ) {
      
      var a = 3;
      console.log( a ); // 3
      console.log( global.a ); // 2
      
  })( window );
  
  console.log( a ); // 2
  ```



* 解决`undefined`标识符的默认值被错误覆盖导致的异常

  将一个参数命名为`undefined`，但是在对应的位置不传入任何值，这样就可以保证在代码块中`undefined`标识符的值真的是`undefined`。

  ```Js
  undefined = true;
  
  (function IIFE( undefined ) {
      
      var a;
      if (a === undefined) {
          console.log("Undefined is safe here!");
      }
  })();
  ```



* 倒置代码的运行顺序，将需要运行的函数放在第二位，在IIFE执行**之后**当做参数传递进去

  函数表达式`def`定义在片段的第二部分，然后当做参数（这个参数也叫做def）被传递进IIFE函数定义的第一部分中。最后，参数def（也就是传递进去的函数）被调用，并将window传入当做global参数的值。

  ```Js
  var a = 2;
  
  (function IIFE( def ) {
      def( window );
  })(function def( global ) {
     
      var a = 3;
      console.log( a ); // 3
      console.log( global.a ); // 2
      
  });
  ```

  

#### 3.4 块作用域

表面上看JavaScript并没有块作用域的相关功能，除非更加深入了解（with、try/catch 、let、const）。

```Js
for (var i = 0; i < 10; i++) {
    console.log( i );
}
```

上述代码中`i`会被绑定在外部作用域（函数或全局）中。

```Js
var foo = true;

if (foo) {
    var bar = foo * 2;
    bar = something( bar );
    console.log( bar );
}
```

上述代码中，当使用var声明变量时，它写在哪里都是一样的，因为它们最终都会属于外部作用域。



##### 3.4.1 with

块作用域的一种形式，用`with`从对象中创建出的作用域仅在**`with`声明中**而非外部作用域中有效。



##### 3.4.2 try/catch

ES3规范中规定try/catch的catch分句会创建一个块作用域，其中声明的变量仅在catch中有效。

```Js
try {
    undefined(); // 执行一个非法操作来强制制造一个异常
}
catch (err) {
    console.log( err ); // 能够正常执行！
}

console.log( err )； // ReferenceError: err not found
```

当同一个作用域中的两个或多个catch分句用同样的标识符名称声明错误变量时，很多静态检查工具还是会发出警告，**实际上这并不是重复定义**，因为所有变量都会安全地限制在块作用域内部。



##### 3.4.3 let

ES6引入了`let`关键字，可以将变量绑定到所在的任意作用域中（通常是`{ .. }`内部），即`let`为其声明的变量隐式地**劫持**了所在的块作用域。

```Js
var foo = true;

if (foo) {
    let bar = foo * 2;
    bar = something( bar );
    console.log( bar );
}

console.log( bar ); // ReferenceError
```

> 存在的问题

用`let`将变量附加在一个已经存在的的块作用域上的行为是隐式的，如果习惯性的移动这些块或者将其包含在其他的块中，可能会导致代码混乱。



> 解决方案

为块作用域显示地创建块。显式的代码优于隐式或一些精巧但不清晰的代码。

```Js
var foo = true;

if (foo) {
    { // <-- 显式的块
        let bar = foo * 2;
        bar = something( bar );
        console.log( bar );
    }
}

console.log( bar ); // ReferenceError
```

在if声明内部显式地创建了一个块，如果需要对其进行重构，整个块都可以被方便地移动而不会对外部if声明的位置和语义产生任何影响。

* 在let进行的声明不会在块作用域中进行提升

  ```Js
  console.log( bar ); // ReferenceError
  let bar = 2;
  ```

  

* 1、垃圾收集

  ```Js
  function process(data) {
      // 在这里做点有趣的事情
  }
  
  var someReallyBigData = { .. };
  
  process( someReallyBigData );
  
  var btn = document.getElementById( "my_button" );
  
  btn.addEventListener( "click", function click(evt) {
      console.log("button clicked");
  }, /*capturingPhase*/false );
  ```

  `click`函数的点击回调并**不需要**`someReallyBigData`。理论上当`process(..)`执行后，在内存中占用大量空间的数据结构就可以被垃圾回收了。但是，由于`click`函数形成了一个覆盖整个作用域的闭包，JS引擎极有可能依然保存着这个结构（取决于具体实现）。

  

* 2、let循环

  ```Js
  for (let i = 0; i < 10; i++) {
      console.log( i );
  }
  
  console.log( i ); // ReferenceError
  ```

  for循环头部的let不仅将i绑定到了for循环的块中，事实上它将其重新绑定到了循环的每一个迭代中，确保使用上一个循环迭代结束时的值重新进行赋值。

  ```Js
  {
      let j;
      for (j = 0; j < 10; j++) {
          let i = j; // 每个迭代重新绑定!
          console.log( i ); 
     	} 
  }
  ```

  

##### 3.4.4 const

ES6引用了`const`，可以创建块作用域变量，但其值是固定的（常量）

```Js
var foo = true;

if(foo) {
    var a = 2;
    const b = 3; // 包含在if中的块作用域常量
    
    a = 3; // 正常!
    b = 4; // 错误!
}

console.log( a ); // 3
console.log( b ); // ReferenceError!
```



### 第4章 提升 

* 任何声明在某个作用域内的变量，都将附属于这个作用域。
* 包括变量和函数在内的所有声明都会在任何代码被执行前首先被处理。
* `var a = 2;`会被看成两个声明，`var a;`和`a = 2;`，第一个声明在**编译阶段**进行，第二个赋值声明会被留在原地等待**执行阶段**。
* 所有的声明（变量和函数）都会被**“移动”**到各自作用域的最顶端，这个过程叫做**提升**
* 只有声明本身会被提升，而包括函数表达式在内的赋值或其他运行逻辑并不会提升。

```Js
a = 2;

var a;

console.log( a ); // 2

---------------------------------------
// 实际按如下形式进行处理
var a; // 编译阶段

a = 2; // 执行阶段

console.log( a ); // 2
```



```Js
console.log( a ); // undefinde

var a = 2;

---------------------------------------
// 实际按如下形式进行处理
var a; // 编译

console.log( a ); // undefinde

a = 2; // 执行
```

* 每个作用域都会进行变量提升

```Js
function foo() {
    var a;
    
    console.log( a ); // undefinde
    
    a = 2;
}

foo();
```

* 函数声明会被提升，但是**函数表达式不会被提升**。

```Js
foo(); // 不是ReferenceError，而是TypeError!

var foo = function bar() {
    // ...
};
```

上面这段程序中，变量标识符`foo()`被提升并分配给所在作用域，因此`foo()`不会导致ReferenceError。此时`foo`并没有赋值（**如果它是一个函数声明而不是函数表达式，那么就会赋值**），`foo()`由于对`undefined`值进行函数调用而导致非法操作，因此抛出`TypeError`异常。

* 即使是具名的函数表达式，名称标识符在赋值之前也无法在所在作用域中使用。

```Js
foo(); // TypeError
bar(); // ReferenceError

var foo = function bar() {
    // ...
};

---------------------------------------
// 实际按如下形式进行处理
var foo;

foo(); // TypeError
bar(); // ReferenceError

foo = function() {
    var bar = ...self...
    // ...
};
```



#### 4.1 函数优先

* 函数声明和变量声明都会被提升，但是，**函数首先被提升，然后才是变量**

```Js
foo(); // 1

var foo;

function foo() {
    console.log( 1 ); 
};

foo = function() {
    console.log( 2 ); 
};

---------------------------------------
// 实际按如下形式进行处理

function foo() { // 函数提升是整体提升，声明 + 赋值
    console.log( 1 ); 
};

foo(); // 1

foo = function() {
    console.log( 2 ); 
};
```

* `var foo`尽管出现在`function foo()...`的声明之前，但它是重复的声明，且函数声明会被提升到普通变量之前，因此被忽略
* 后面出现的函数声明可以**覆盖**前面的。

```Js
foo(); // 3

function foo() {
    console.log( 1 ); 
};

var foo = function() {
    console.log( 2 ); 
};

function foo() {
    console.log( 3 ); 
};
```

* 一个普通块内部的函数声明通常会被提升到所在作用域的顶部，不会被条件判断所控制。**尽量避免在普通块内部声明函数**。

```Js
foo(); // "b"

var a = true;
if (a) {
    function foo() { console.log( "a" ); };
}
else {
    function foo() { console.log( "b" ); };
}
```



### 第5章 作用域闭包

#### 5.1 闭包

* 当函数可以记住并访问所在的词法作用域，即使函数名是在当前词法作用域之外执行，这时就产生了闭包。

```Js
function foo() {
    var a = 2;
    
    function bar() {
		console.log( a );
    }
    
    return bar;
}

var baz = foo();

baz(); // 2 ---- 这就是闭包的效果
```

`bar()`在自己定义的词法作用域以外的地方执行。

`bar()`拥有覆盖`foo()`内部作用域的闭包，使得该作用域能够一直存活，以供`bar()`在之后任何时间进行引用，不会被垃圾回收器回收

* `bar()`持有对`foo()`内部作用域的引用，这个引用就叫做闭包。



```Js
// 对函数类型的值进行传递
function foo() {
    var a = 2;
    
    function baz() {
		console.log( a ); // 2
    }
    
    bar( baz );
}

function bar(fn) {
    fn(); // 这就是闭包
}

foo();
```

* 把内部函数`baz`传递给`bar`，当调用这个内部函数时（现在叫做`fn`），它覆盖的`foo()`内部作用域的闭包就形成了，因为它能够访问a。



```Js
// 间接的传递函数
var fn;

function foo() {
    var a = 2;
    
    function baz() {
		console.log( a ); 
    }
    
    fn = baz; // 将baz分配给全局变量
}

function bar() {
    fn(); // 这就是闭包
}

foo();
bar(); // 2
```

* 将内部函数传递到所在的词法作用域以外，它都会持有对原始定义作用域的引用，无论在何处执行这个函数都会使用闭包。



```Js
function wait(message) {
    
    setTimeout( function timer() {
        console.log( message );
    }, 1000 );
}

wait( "Hello, closure!" );
```

* 在引擎内部，内置的工具函数`setTimeout(..)`持有对一个参数的引用，这里参数叫做timer，引擎会调用这个函数，而词法作用域在这个过程中保持完整。**这就是闭包**
* 定时器、事件监听器、Ajax请求、跨窗口通信、Web Workers或者任何其他的异步（或者同步）任务中，只要使用了**回调函数**，实际上就是在使用**闭包**！



```Js
// 典型的闭包例子：IIFE
var a = 2;

(function IIFE() {
    console.log( a );
})();
```



#### 5.2 循环和闭包

```Js
for (var i = 1; i <= 5; i++) {
    setTimeout( function timer() {
        console.log( i );
    }, i * 1000 );
}

//输入五次6
```

* 延迟函数的回调会在循环结束时才执行，输出显示的是循环结束时`i`的最终值。
* 尽管循环中的五个函数是在各个迭代中分别定义的，但是它们都被封闭在一个共享的全局作用域中，因此实际上只有一个`i`



尝试方案1：使用IIFE增加更多的闭包作用域

```Js
for (var i = 1; i <= 5; i++) {
    (function() {
        setTimeout( function timer() {
        	console.log( i );
    	}, i * 1000 );
    })();
}

//失败，因为IIFE作用域是空的，需要包含一点实质内容才可以使用
```



尝试方案2：IIFE增加变量

```Js
for (var i = 1; i <= 5; i++) {
    (function() {
        var j = i;
        setTimeout( function timer() {
        	console.log( j );
    	}, j * 1000 );
    })();
}

// 正常工作
```



尝试方案3：改进型，将`i`作为参数传递给IIFE函数

```Js
for (var i = 1; i <= 5; i++) {
    (function(j) {
        setTimeout( function timer() {
        	console.log( j );
    	}, j * 1000 );
    })( i );
}

// 正常工作
```



##### 5.2.1 块作用域和闭包

* `let`可以用来劫持块作用域，并且在这个块作用域中声明一个变量。
* **本质上这是将一个块转换成一个可以被关闭的作用域**

```Js
for (var i = 1; i <= 5; i++) {
    let j = i; // 闭包的块作用域！
    setTimeout( function timer() {
        console.log( j );
    }, j * 1000 );
}

// 正常工作
```



* `for`循环头部的`let`声明会有一个特殊的行为。变量在循环过程中不止被声明一次，**每次迭代**都会声明。随后的每个迭代都会使用上一个迭代结束时的值来初始化这个变量。

> 上面这句话参照3.4.3–---2.let循环，即以下

```Js
{
    let j;
    for (j = 0; j < 10; j++) {
        let i = j; // 每个迭代重新绑定!
        console.log( i ); 
   	} 
}
```



循环改进：

```Js
for (let i = 1; i <= 5; i++) {
    setTimeout( function timer() {
        console.log( i );
    }, i * 1000 );
}

// 正常工作
```



#### 5.3 模块

模块模式需要具备两个必要条件：

* 必须有外部的封闭函数，该函数必须至少**被调用一次**（每次调用都会创建一个新的模块实例，可以通过IIFE实现单例模式）
* 封闭函数必须**返回至少一个内部函数**，这样内部函数才能在私有作用域中形成闭包，并且可以访问或者修改私有的状态。

```Js
function CoolModule() {
    var something = "cool";
    var another = [1, 2, 3];
    
    function doSomething() {
        console.log( something );
    }
    
    function doAnother() {
        console.log( another.join( " ! ") );
    }
    
    return {
        doSomething: doSomething,
        doAnother: doAnother
    }
}

var foo = CoolModule();

foo.doSomething(); // cool
foo.doAnother(); // 1 ! 2 ! 3

// 1、必须通过调用CoolModule()来创建一个模块实例
// 2、CoolModule()返回一个对象字面量语法{ key: value, ... }表示的对象，对象中含有对内部函数而不是内部数据变量的引用。内部数据变量保持隐藏且私有的状态。
```

* 使用IIFE实现单例模式

**立即**调用这个函数并将返回值直接赋予给单例的模块标识符foo。

```Js
var foo = (function CoolModule() {
    var something = "cool";
    var another = [1, 2, 3];
    
    function doSomething() {
        console.log( something );
    }
    
    function doAnother() {
        console.log( another.join( " ! ") );
    }
    
    return {
        doSomething: doSomething,
        doAnother: doAnother
    }
})();

foo.doSomething(); // cool
foo.doAnother(); // 1 ! 2 ! 3
```



#### 5.5.1 现代的模块机制

大多数模块依赖加载器/管理器本质上是将这种模块定义封装进一个友好的API。

```Js
var MyModules = (function Manager() {
    var modules = {};
    
    function define(name, deps, impl) {
        for (var i = 0; i < deps.length; i++ ) {
			deps[i] = modules[deps[i]];
        }
        modules[name] = impl.apply( impl, deps ); // 核心，为了模块的定义引用了包装函数(可以传入任何依赖)，并且将返回值(模块的API)，储存在一个根据名字来管理的模块列表中。
    }
    
    function get(name) {
        return modules[name];
    }
    
    return {
        define: define,
        get: get
    };
    
})();
```



使用上面的函数来定义模块：

```Js
MyModules.define( "bar", [], function() {
    function hello(who) {
        return "Let me introduct: " + who;
    }
    
    return {
        hello: hello
    };
} );

MyModules.define( "foo", ["bar"], function(bar) {
    var hungry = "hippo";
    
    function awesome() {
        console.log( bar.hello( hungry ).toUpperCase() );
    }
    
    return {
        awesome: awesome
    };
} );

var bar = MyModules.get( "bar" );
var foo = MyModules.get( "foo" );

console.log(
	bar.hello( "hippo" );
) // Let me introduct: hippo

foo.awesome(); // LET ME INTRODUCT: HIPPO
```



#### 5.5.2 未来的模块机制

在通过模块系统进行加载时，ES6会将文件当做独立的模块来处理。每个模块都可以导入其他模块或特定的API成员，同样可以导出自己的API成员。

ES6模块没有“行内”格式，必须被定义在独立的文件中（一个文件一个模块）

* 基于函数的模块不能被静态识别（编译器无法识别），只有在运行时才会考虑API语义，因此可以在运行时修改一个模块的API。
* ES6模块API是静态的（API模块不会在运行时改变），会在编译期检查对导入模块的API成员的引用是否真实存在。

```Js
// bar.js

function hello(who) {
    return "Let me introduct: " + who;
}

export hello;


// foo.js
// 仅从“bar”模块导入hello()
import hello from "bar";

var hungry = "hippo";

function awesome() {
    console.log(
    	hello( hungry ).toUpperCase();
    );
}

export awesome;

// baz.js
// 导入完整的“foo”和”bar“模块
module foo from "foo";
module bar from "bar";

console.log(
	bar.hello( "rhino")
); // Let me introduct: rhino

foo.awesome(); // LET ME INTRODUCT: HIPPO
```

* `import`：将一个模块中的一个或多个API导入到当前作用域中，并分别绑定在一个变量上
* `module`：将整个模块的API导入并绑定到一个变量上。
* `export`：将当前模块的一个标识符（变量、函数）导出为公共API



### 附录A 动态作用域

* 词法作用域是在写代码或者定义时确定的，关注函数**在何处声明**，作用域链基于代码嵌套。
* 动态作用域是在运行时确定的（**this也是**），关注函数**从何处调用**，作用域链基于调用栈。
* JavaScript并**不具备**动态作用域，它只有词法作用域。但是`this`机制某种程度上很像动态作用域。

```Js
// 词法作用域，关注函数在何处声明，a通过RHS引用到了全局作用域中的a
function foo() {
    console.log( a ); // 2
}

function bar() {
    var a = 3;
    foo();
}

var a = 2;
bar();

-----------------------------
// 动态作用域，关注函数从何处调用，当foo()无法找到a的变量引用时，会顺着调用栈在调用foo()的地方查找a
function foo() {
    console.log( a ); // 3(不是2！)
}

function bar() {
    var a = 3;
    foo();
}

var a = 2;
bar();
```



###附录B 块作用域的替代方案

ES3开始，JavaScript中就有了块作用域，包括with和catch分句。

```js
// ES6环境
{
    let a = 2;
    console.log( a ); // 2
}

console.log( a ); // ReferenceError
```

上述代码在ES6环境中可以正常工作，但是在ES6之前的环境中如何实现呢？

答案是**使用catch分句**，这是ES6中大部分功能迁移的首选方式。

```js
try {
    throw 2;
} catch (a) {
    console.log( a ); // 2
}

console.log( a ); // ReferenceError
```



#### B.1 Traceur

```Js
// 代码转换成如下形式
{
    try {
        throw undefined;
    } catch (a) {
        a = 2;
        console.log( a ); // 2
    }
}

console.log( a ); // ReferenceError
```



#### B.2 隐式和显式作用域 

`let`声明会创建一个显式的作用域并与其进行绑定，而不是隐式地劫持一个已经存在的作用域（对比前面的`let`定义）。

```Js
let (a = 2) {
    console.log( a ); // 2
}

console.log( a ); // ReferenceError
```



存在的问题：

`let`声明不包含在ES6中，Traceur编译器也不接受这种代码

* 方案一：使用合法的ES6语法并且在代码规范上做一些妥协

```Js
/*let*/ { let a = 2;
    console.log( a );
}

console.log( a ); // ReferenceError
```



* 方案二：使用let-er工具，生成完全标准的ES6代码，不会生成通过try/catch进行hack的ES3替代方案

```Js
{
    let a = 2;
    console.log( a );
}

console.log( a ); // ReferenceError
```



#### B.3 性能

* try/catch的性能的确很糟糕，但技术层面上没有合理的理由来说明try/catch必须这么慢，或者会一直慢下去。
* IIFE和try/catch不是完全等价的，因为如果把一段代码中的任意一部分拿出来用函数进行包裹，会改变这段代码的含义，其中的this、return、break和continue都会发生变化。IIFE并不是一个普适的方案，只适合在某些情况下进行手动操作。



## 第二部分 this和对象原型

### 第1章 关于this

* `this`既不指向函数自身也不指向函数的词法作用域
* `this`实际上是在函数被调用时发生的绑定，它指向什么完全取决于函数在哪里被调用。this和词法作用域是不一样的，不能混合使用。
* 当一个函数被调用时，会创建一个活动记录（执行上下文）。这个记录会包含函数在哪里被调用（调用栈）、函数的调用方式、传入的参数等信息。**this就是这个记录的一个属性**，会在函数执行的过程中用到。

```Js
// 使用this，提供一种优雅的方式隐式“传递”一个对象引用。API设计更加简洁并且易于复用。
function identify() {
    return this.name.toUpperCase();
}

function speak() {
    var greeting = "Hello, I'm " + identify.call( this );
    console.log( greeting );
}

var me = {
    name: "Kyle"
};

var you = {
    name: "Reader"
};

identify.call( me ); // KYLE
identify.call( you ); // READER

speak.call( me ); // Hello, I'm KYLE
speak.call( you ); // Hello, I'm READER

// -----------------------------------------------
// 词法作用域方案，显式传递上下文对象方式，会让代码变得越来越混乱
function identify(context) {
    return context.name.toUpperCase();
}

function speak(context) {
    var greeting = "Hello, I'm " + identify( context );
    console.log( greeting );
}

identify( you ); // READER
speak.call( me ); // Hello, I'm KYLE
```



### 第2章 this全面解析

#### 2.1 调用位置

调用位置就是函数在代码中被调用的位置（而不是声明的位置）。

查找方法：

* 分析调用栈：调用位置就是当前正在执行的函数的前一个调用中

  ```Js
  function baz() {
      // 当前调用栈是：baz
      // 因此，当前调用位置是全局作用域
      
      console.log( "baz" );
      bar(); // <-- bar的调用位置
  }
  
  function bar() {
      // 当前调用栈是：baz --> bar
      // 因此，当前调用位置在baz中
      
      console.log( "bar" );
      foo(); // <-- foo的调用位置
  }
  
  function foo() {
      // 当前调用栈是：baz --> bar --> foo
      // 因此，当前调用位置在bar中
      
      console.log( "foo" );
  }
  
  baz(); // <-- baz的调用位置
  ```

  

* 使用开发者工具得到调用栈：

  设置断点或者插入`debugger;`语句，运行时调试器会在那个位置暂停，同时展示当前位置的函数调用列表，这就是调用栈。找到栈中的第二个元素，这就是真正的调用位置。



#### 2.2 绑定规则

##### 2.2.1 默认绑定

* **独立函数调用**，可以把默认绑定看作是无法应用其他规则时的默认规则，this指向**全局对象**。
* **严格模式**下，不能将全局对象用于默认绑定，this会绑定到`undefined`。只有函数**运行**在非严格模式下，默认绑定才能绑定到全局对象。在严格模式下**调用**函数则不影响默认绑定。

```Js
function foo() { // 运行在严格模式下，this会绑定到undefined
    "use strict";
    
    console.log( this.a );
}

var a = 2;

// 调用
foo(); // TypeError: this is undefined

// --------------------------------------

function foo() { // 运行
    console.log( this.a );
}

var a = 2;

(function() { // 严格模式下调用函数则不影响默认绑定
    "use strict";
    
    foo(); // 2
})();
```



##### 2.2.2 隐式绑定

当函数引用有**上下文对象**时，隐式绑定规则会把函数中的this绑定到这个上下文对象。对象属性引用链中只有上一层或者说最后一层在调用中起作用。

```Js
function foo() {
    console.log( this.a );
}

var obj = {
    a: 2,
    foo: foo
};

obj.foo(); // 2
```



> 隐式丢失

被隐式绑定的函数特定情况下会丢失绑定对象，应用默认绑定，把this绑定到全局对象或者undefined上。

```Js
// 虽然bar是obj.foo的一个引用，但是实际上，它引用的是foo函数本身。
// bar()是一个不带任何修饰的函数调用，应用默认绑定。
function foo() {
    console.log( this.a );
}

var obj = {
    a: 2,
    foo: foo
};

var bar = obj.foo; // 函数别名

var a = "oops, global"; // a是全局对象的属性

bar(); // "oops, global"
```



参数传递就是一种隐式赋值，传入函数时也会被隐式赋值。回调函数丢失this绑定是非常常见的。

```Js
function foo() {
    console.log( this.a );
}

function doFoo(fn) {
    // fn其实引用的是foo
    
    fn(); // <-- 调用位置！
}

var obj = {
    a: 2,
    foo: foo
};

var a = "oops, global"; // a是全局对象的属性

doFoo( obj.foo ); // "oops, global"

// ----------------------------------------

// JS环境中内置的setTimeout()函数实现和下面的伪代码类似：
function setTimeout(fn, delay) {
    // 等待delay毫秒
    fn(); // <-- 调用位置！
}
```



##### 2.2.3 显式绑定

通过`call(..)` 或者 `apply(..)`方法。第一个参数是一个对象，在调用函数时将这个对象绑定到this。因为直接指定this的绑定对象，称之为显示绑定。

```Js
function foo() {
    console.log( this.a );
}

var obj = {
    a: 2
};

foo.call( obj ); // 2  调用foo时强制把foo的this绑定到obj上
```



显示绑定无法解决丢失绑定问题。

解决方案：

* 1、硬绑定

创建函数bar()，并在它的内部手动调用foo.call(obj)，强制把foo的this绑定到了obj。

```Js
function foo() {
    console.log( this.a );
}

var obj = {
    a: 2
};

var bar = function() {
    foo.call( obj );
};

bar(); // 2
setTimeout( bar, 100 ); // 2

// 硬绑定的bar不可能再修改它的this
bar.call( window ); // 2
```



典型应用场景是创建一个包裹函数，负责接收参数并返回值。

```Js
function foo(something) {
    console.log( this.a, something );
    return this.a + something;
}

var obj = {
    a: 2
};

var bar = function() {
    return foo.apply( obj, arguments );
};

var b = bar( 3 ); // 2 3
console.log( b ); // 5
```



创建一个可以重复使用的辅助函数。

```Js
function foo(something) {
    console.log( this.a, something );
    return this.a + something;
}

// 简单的辅助绑定函数
function bind(fn, obj) {
    return function() {
        return fn.apply( obj, arguments );
    }
}

var obj = {
    a: 2
};

var bar = bind( foo, obj );

var b = bar( 3 ); // 2 3
console.log( b ); // 5
```



ES5内置了`Function.prototype.bind`，bind会返回一个硬绑定的新函数，用法如下。

```Js
function foo(something) {
    console.log( this.a, something );
    return this.a + something;
}

var obj = {
    a: 2
};

var bar = foo.bind( obj );

var b = bar( 3 ); // 2 3
console.log( b ); // 5
```



* 2、API调用的“上下文”

JS许多内置函数提供了一个可选参数，被称之为“上下文”（context），其作用和`bind(..)`一样，确保回调函数使用指定的this。这些函数实际上通过`call(..)`和`apply(..)`实现了显式绑定。

```js
function foo(el) {
	console.log( el, this.id );
}

var obj = {
    id: "awesome"
}

// 调用foo(..)时把this绑定到obj
[1, 2, 3].forEach( foo, obj );
// 1 awesome 2 awesome 3 awesome
```



##### 2.2.4 new绑定

* 在JS中，`构造函数`只是使用`new`操作符时被调用的`普通`函数，他们不属于某个类，也不会实例化一个类。
* 包括内置对象函数（比如`Number(..)`）在内的所有函数都可以用new来调用，这种函数调用被称为构造函数调用。
* 实际上并不存在所谓的“构造函数”，只有对于函数的“**构造调用**”。



使用new来调用函数，或者说发生构造函数调用时，会自动执行下面的操作。

* 1、创建（或者说构造）一个新对象。
* 2、这个新对象会被执行[[Prototype]]连接。
* 3、这个新对象会绑定到函数调用的this。
* 4、如果函数没有返回其他对象，那么new表达式中的函数调用会自动返回这个新对象。



使用`new`来调用`foo(..)`时，会构造一个新对象并把它（bar）绑定到`foo(..)`调用中的this。

```Js
function foo(a) {
    this.a = a;
}

var bar = new foo(2); // bar和foo(..)调用中的this进行绑定
console.log( bar.a ); // 2
```



#### 2.3 优先级

```Flow
st=>start: Start
e=>end: End
cond1=>condition: new绑定
op1=>operation: this绑定新创建的对象，
				var bar = new foo()
				
cond2=>condition: 显示绑定
op2=>operation: this绑定指定的对象，
				var bar = foo.call(obj2)
				
cond3=>condition: 隐式绑定
op3=>operation: this绑定上下文对象，
				var bar = obj1.foo()
				
op4=>operation: 默认绑定
op5=>operation: 函数体严格模式下绑定到undefined，
				否则绑定到全局对象，
				var bar = foo()

st->cond1
cond1(yes)->op1->e
cond1(no)->cond2
cond2(yes)->op2->e
cond2(no)->cond3
cond3(yes)->op3->e
cond3(no)->op4->op5->e
```



在`new`中使用硬绑定函数的目的是预先设置函数的一些参数，这样在使用new进行初始化时就可以只传入其余的参数（柯里化）。

```Js
function foo(p1, p2) {
    this.val = p1 + p2;
}

// 之所以使用null是因为在本例中我们并不关心硬绑定的this是什么
// 反正使用new时this会被修改
var bar = foo.bind( null, "p1" );

var baz = new bar( "p2" );

baz.val; // p1p2
```



#### 2.4 绑定例外

##### 2.4.1 被忽略的this

把null或者undefined作为this的绑定对象传入call、apply或者bind，这些值在调用时会被忽略，实际应用的是默认规则。



下面两种情况下会传入null

* 使用`apply(..)`来“展开”一个数组，并当作参数传入一个函数
* `bind(..)`可以对参数进行柯里化（预先设置一些参数）

```js
function foo(a, b) {
    console.log( "a:" + a + "，b:" + b );
}

// 把数组”展开“成参数
foo.apply( null, [2, 3] ); // a:2，b:3

// 使用bind(..)进行柯里化
var bar = foo.bind( null, 2 );
bar( 3 ); // a:2，b:3 
```



总是传入null来忽略this绑定可能产生一些副作用。如果某个函数确实使用了this，那默认绑定规则会把this绑定到全局对象中。

> 更安全的this

安全的做法就是传入一个特殊的对象（空对象），把this绑定到这个对象不会对你的程序产生任何副作用。

JS中创建一个空对象最简单的方法是Object.create(null)，这个和{}很像，但是并不会创建`Object.prototype`这个委托，所以比{}更空。

```Js
function foo(a, b) {
    console.log( "a:" + a + "，b:" + b );
}

// 我们的空对象
var ø = Object.create( null );

// 把数组”展开“成参数
foo.apply( ø, [2, 3] ); // a:2，b:3

// 使用bind(..)进行柯里化
var bar = foo.bind( ø, 2 );
bar( 3 ); // a:2，b:3 
```



##### 2.4.2 间接引用

间接引用下，调用这个函数会应用默认绑定规则。间接引用最容易在赋值时发生。

```Js
// p.foo = o.foo的返回值是目标函数的引用，所以调用位置是foo()而不是p.foo()或者o.foo()
function foo() {
    console.log( this.a );
}

var a = 2;
var o = { a: 3, foo: foo };
var p = { a: 4};

o.foo(); // 3
(p.foo = o.foo)(); // 2
```



##### 2.4.3 软绑定

* 硬绑定可以把this强制绑定到指定的对象（new除外），防止函数调用应用默认绑定规则。但是会降低函数的灵活性，使用硬绑定之后就无法使用隐式绑定或者显式绑定来修改this。
* **如果给默认绑定指定一个全局对象和undefined以外的值**，那就可以实现和硬绑定相同的效果，同时保留隐式绑定或者显示绑定修改this的能力。



```js
// 默认绑定规则，优先级排最后
// 如果this绑定到全局对象或者undefined，那就把指定的默认对象obj绑定到this,否则不会修改this
if(!Function.prototype.softBind) {
    Function.prototype.softBind = function(obj) {
        var fn = this;
        // 捕获所有curried参数
        var curried = [].slice.call( arguments, 1 ); 
        var bound = function() {
            return fn.apply(
            	(!this || this === (window || global)) ? 
                	obj : this,
                curried.concat.apply( curried, arguments )
            );
        };
        bound.prototype = Object.create( fn.prototype );
        return bound;
    };
}
```



使用：软绑定版本的foo()可以手动将this绑定到obj2或者obj3上，但如果应用默认绑定，则会将this绑定到obj。

```Js
function foo() {
    console.log("name:" + this.name);
}

var obj = { name: "obj" },
    obj2 = { name: "obj2" },
    obj3 = { name: "obj3" };

// 默认绑定，应用软绑定，软绑定把this绑定到默认对象obj
var fooOBJ = foo.softBind( obj );
fooOBJ(); // name: obj 

// 隐式绑定规则
obj2.foo = foo.softBind( obj );
obj2.foo(); // name: obj2 <---- 看！！！

// 显式绑定规则
fooOBJ.call( obj3 ); // name: obj3 <---- 看！！！

// 绑定丢失，应用软绑定
setTimeout( obj2.foo, 10 ); // name: obj
```



#### 2.5 this词法

ES6新增一种特殊函数类型：箭头函数，箭头函数无法使用上述四条规则，而是根据外层（函数或者全局）作用域（词法作用域）来决定this.



* foo()内部创建的箭头函数会捕获调用时foo()的this。由于foo()的this绑定到obj1，bar(引用箭头函数)的this也会绑定到obj1，**箭头函数的绑定无法被修改**(new也不行)。

```Js
function foo() {
    // 返回一个箭头函数
    return (a) => {
        // this继承自foo()
        console.log( this.a );
    };
}

var obj1 = {
    a: 2
};

var obj2 = {
    a: 3
}

var bar = foo.call( obj1 );
bar.call( obj2 ); // 2，不是3！
```



ES6之前和箭头函数类似的模式，采用的是词法作用域取代了传统的this机制。

```Js
function foo() {
    var self = this; // lexical capture of this
    setTimeout( function() {
        console.log( self.a ); // self只是继承了foo()函数的this绑定
    }, 100 );
}

var obj = {
    a: 2
};

foo.call(obj); // 2
```



代码风格统一问题：如果既有this风格的代码，还会使用 `seft = this` 或者箭头函数来否定this机制。

* 只使用词法作用域并完全抛弃错误this风格的代码；
* 完全采用this风格，在必要时使用`bind(..)`，尽量避免使用 `self = this` 和箭头函数。



### 第3章 对象

#### 3.1 语法

文字形式和构造形式生成的对象是一样的。

* 文字（声明）形式

  ```Js
  var myObj = {
      key: value
      // ...
  }
  ```

  

* 构造形式

  ```Js
  var myObj = new Object();
  myObj.key = value;
  ```

  

#### 3.2 类型

* 基本数据类型：`string`、`number`、`boolean`、`null`、`undefined`、`symbol`

  * 上述6种数据类型本身不是对象，”JS中万物皆是对象“的说法是错误的
  * `typeof null`返回字符串”object“，实际上，`null`本身是基本数据类型，这是语言本身的一个bug。(原理是这样：**不同的对象在底层都表示为二进制，在JS中二进制前三位都为0的话会被判断为object类型，null的二进制表示是全0，自然前三位也是0，所以执行typeof时会返回”object“**)

  

* 引用数据类型：统称为`object`类型，称作对象子类型（内置对象），细分有`String`、`Number`、`Boolean`、`Object`、`Function`、`Array`、`Date`、`RegExp`、`Error`



```Js
var strPrimitive = "I am a string";
typeof strPrimitive; // "string"
strPrimitive instanceof String; // false

var strObject = new String("I am a string");
typeof strObject; // "object"
strObject instanceof String; // true

// 检查sub-type对象
// 可以认为子类型在内部借用了Object中的toString()方法
Object.prototype.toString.call( strObject ); // [object String]
```

上述strPrimitive并不是一个对象，只是一个字面量，并且是一个不可变的值。如果需要执行一些操作，比如获取长度、访问其中某个字符等，那需要转换成String对象。

* JS引擎会自动把字面量转换成String对象，所以可以访问属性和方法。数值字面量和布尔字面量同理。

```js
var strPrimitive = "I am a string";

console.log( strPrimitive.length ); // 13

console.log( strPrimitive.charAt( 3 ) ); // "m"
```



* null和undefined没有对应的构造形式，Date只有构造，没有文字形式。
* Object、Array、Function和RegExp（正则表达式），无论使用文字形式还是构造形式，它们都是对象，不是字面量。



#### 3.3 内容

* 对象的内容是由一些存储在特定命名位置的值（任意类型的）组成的，称之为**属性**。
* 引擎内部，属性的值一般不保存在对象容器内部，存储在对象容器内部的是这些属性的名称，通过引用指向这些值真正的存储位置。



* .a语法称为”属性访问“，["a"]语法称为”键访问“。它们访问的是同一个位置。

```Js
var myObject = {
    a: 2
};

myObject.a; // 2       属性 <----> 值
myObject["a"]; // 2    键  <----> 值
```



* [".."]语法使用字符串来访问属性，可以在程序中构造这个字符串

```Js
var myObject = {
    a: 2
};

var idx;

if (true) {
    idx = "a";
}

// 之后

console.log( myObject[idx] ); // 2
```



* 对象属性名永远是字符串，如果是其他类型，则会转换为一个字符串，数字也不例外。

```Js
var myObject = { };

myObject[true] = "foo";
myObject[3] = "bar";
myObject[myObject] = "baz";

myObject["true"]; // "foo";
myObject["3"]; // "bar";
myObject["[object Object]"]; // "baz";
```



##### 3.3.1 可计算属性名

ES6增加可计算属性名，可以在文字形式中使用[ ]包裹一个表达式来当做属性名

```Js
var prefix = "foo";

var myObject = {
    [prefix + "bar"]: "hello",
    [prefix + "baz"]: "world"
};

myObject["foobar"]; // hello
myObject["foobaz"]; // world
```



##### 3.3.2 属性与方法

技术角度来说，函数永远不会”属于“一个对象。有些函数具有this引用，有时候这些this确实会指向调用位置的对象引用。但是这是this在运行时根据调用位置动态绑定的，本质上并没有把一个函数变成一个”方法“，函数和对象的关系最多只能是间接关系。

```Js
function foo() {
    console.log( "foo" );
}

var someFoo = foo; // 对foo的变量引用

var myObject = {
    someFoo: foo
};

// 对同一个对象的不同引用
foo; // function foo(){..}

someFoo; // function foo(){..}

myObject.someFoo; // function foo(){..}
```



##### 3.3.3 数组

* 数组期望的是数值下标，索引是非负数。

```Js
var myArray = [ "foo", 42, "bar" ];

myArray.length; // 3

myArray[0]; // "foo"

myArray[2]; // "bar"
```



* 数组也是对象，可以给数组添加属性。（数组的length值并未发生变化）

```Js
var myArray = [ "foo", 42, "bar" ];

myArray.baz = "baz";

myArray.length; // 3

myArray.baz; // "baz"
```



* 如果添加的属性名”看起来“像一个数字，那它会变成一个数值下标。会修改数组的内容而不是添加一个属性。

```Js
var myArray = [ "foo", 42, "bar" ];

myArray["3"] = "baz";

myArray.length; // 4

myArray[3]; // "baz"
```



##### 3.3.4 复制对象

```Js
function anotherFunction() { /*..*/ }

var anotherObject = {
    c: true
};

var anotherArray = [];

var myObject = {
    a: 2,
    b: anotherObject, // 引用，不是复本！
    c: anotherArray, // 另一个引用！
    d: anotherFunction
}

anotherArray.push( anotherObject, myObject );
```

* 对于**浅拷贝**（浅复制）：新对象中的a是复制的，但是b、c、d只是三个引用，和旧对象中是一样的。

  * `Object.assign(..)`，ES6新增，第一个参数是目标对象，之后跟一个或多个源对象。遍历所有源对象的所有**可枚举**（enumerable）的**自有键**（owned key）,并把它们复制（使用=操作符赋值）到目标对象，最后返回目标对象。

    ```Js
    var newObj = Object.assign( {}, myObject );
    
    newObj.a; // 2
    newObj.b === anotherObject; // true
    newObj.c === anotherArray; // true
    newObj.d === anotherFunction; // true
    ```

  

* 对于**深拷贝**（深复制）：除了复制myObject以外，还会复制anotherObject和anotherArray，问题在于anotherArray还引用了anotherObject和myObject，这样就由于循环引用导致死循环。

  * `JSON安全`（可以被序列化为一个JSON字符串并且可以根据这个字符串解析出一个结构和值完全一样的对象）

    ```Js
    var newObj = JSON.parse( JSON.stringify( someObj ) );
    ```

    

##### 3.3.5 属性描述符

ES5开始，所有的属性都具备了属性描述符（数据描述符）。

* `Object.getOwnPropertyDescriptor(..)`：获取对象属性对应的属性描述符

```Js
var myObject = {
    a: 2
};

Object.getOwnPropertyDescriptor( myObject, "a");
// {
// 		value: 2,
// 		writable: true,		// 可写
// 		enumerable: true,	// 可枚举
// 		configurable: true	// 可配置
// }
```



* `Object.defineProperty(..)`：添加新属性或者修改已有属性（configurable为true）

```Js
var myObject = {};

Object.defineProperty( myObject, "a", {
    value: 2,
    writable: true,		
	enumerable: true,	
	configurable: true	
});

myObject.a; // 2
```



* 1、`Writable`：属性的值是否可修改。

  * false时，对于属性值的修改静默失败（silently failed）。严格模式下，产生TypeError错误。

  

* 2、`configurable`：属性是否可配置。

  * true时，使用`defineProperty(..)`修改属性描述符。
  * false时，尝试使用`defineProperty(..)`会产生TypeError错误（无论是不是严格模式）。改成false是单向操作，无法撤销！
  * false时，还是可以把`writable`的状态由true改为false，但是无法由false改为true。
  * false时，禁止删除这个属性。`delete myObject.a`会静默失败。

  

* 3、`enumerable`：属性是否会出现在对象的属性枚举中，比如说`for..in`循环

  

##### 3.3.6 不变性

实现属性或者对象不可改变的方法有如下多种，但是所有的方法创建的都是**浅不变性**，只会影响目标对象和它的直接属性。



* 1、对象常量

  `writable:false`和`configurable:false`可以创建常量属性（不可修改、重定义或者删除）。

  ```Js
  var myObject = {};
  
  Object.defineProperty( myObject, "FAVORITE_NUMBER", {
      value: 42,
      writable: false,
      configurable: false
  } );
  ```



* 2、禁止扩展

  `Object.preventExtensions(..)`，禁止一个对象添加新属性并且保留已有属性。

  非严格模式下，创建属性b会静默失败。严格模式下，抛出TypeError错误。

  ```Js
  var myObject = {
      a:2
  };
  
  Object.preventExtensions( myObject );
  
  myObject.b = 3;
  myObject.b; // undefined
  ```

  

* 3、密封

  `Object.seal(..)`，密封之后不仅不能添加新属性，也不能重新配置或者删除任何现有属性（但是可以修改属性的值）。

  内部会调用`Object.preventExtensions(..)`并把所有现有属性标记为`configurable:false`。



* 4、冻结

  `Object.freeze(..)`，禁止对于对象本身及其任意直接属性的修改（引用对象不受影响）。

  内部调用`Object.seal(..)`并把所有现有属性标记为`writable:false`。这样就无法修改属性的值。



* 5、深度冻结

  在这个对象上调用`Object.freeze(..)`，然后遍历它引用的所有对象并在这些对象上调用`Object.freeze(..)`。但是这样有可能会在无意中冻结其他（共享）对象。



 ##### 3.3.7 [[Get]]

```Js
var myObject = {
    a: 2
};

myObject.a; // 2
```

* 对象默认的[[Get]]操作可以控制属性值的获取

* `myObject.a`在myObject上实现了[[Get]]操作（有点像函数调用：`[[Get]]()`）。

```Flow
st=>start: Start
e=>end: End
op1=>operation: myObject.a执行[[Get]]操作
cond1=>condition: 对象中是否有名称相同的属性
op2=>operation: 查找成功，返回属性值
op3=>operation: 没有找到，遍历可能存在的[[Prototype]]链
cond2=>condition: 原型链是否查找成功
op4=>operation: 最终没有找到，返回undefined
st->op1->cond1
cond1(yes)->op2->e
cond1(no)->op3->cond2
cond2(yes)->op2
cond2(no)->op4->e
```

> 变量访问和对象属性访问的区别

```Flow
st=>start: Start
e=>end: End
cond1=>condition: 变量访问
op1=>operation: 查找词法作用域
op11=>operation: 对象属性访问
op2=>operation: [[Get]]操作
cond3=>condition: 是否查找成功
op3=>operation: 查找成功，返回变量值
op4=>operation: 查找失败，抛出
				ReferenceError异常
cond4=>condition: 是否查找成功
op5=>operation: 查找成功，返回属性值
op6=>operation: 查找失败，
				返回undefined

st->cond1
cond1(yes)->op1->cond3
cond1(no)->op11->op2->cond4
cond3(yes)->op3->e
cond3(no)->op4->e
cond4(yes)->op5->e
cond4(no)->op6->e
```

* 由于仅根据返回值无法判断出到底变量的值为undefined还是变量不存在，所以[[Get]]操作返回了undefined

```Js
var myObject = {
    a: undefined
};

myObject.a; // undefined

myObject.b; // undefined
```



##### 3.3.8 [[Put]]

* 误解：可能会认为给对象的属性赋值会触发[[Put]]来设置或者创建这个属性。实际情况并不完全如此。
* [[Put]]被触发时，实际行为取决于很多因素，包括对象中是否已经存在这个属性（这是最重要的因素）。
* 对象默认的[[Put]]操作可以控制属性值的设置

```Flow
st=>start: Start
e=>end: End
cond1=>condition: 属性是否存在
op11=>operation: 请看第5章[[Prototype]]
cond2=>condition: 是否是访问描述符
					(参见3.3.9)
op1=>operation: 存在setter就调用setter
cond3=>condition: 数据描述符中
				  writable为false？
op2=>operation: 非严格模式下静默失败，
				严格模式下抛出TypeError异常
op3=>operation: 将该值设置为属性的值

st->cond1
cond1(yes)->cond2
cond1(no)->op11
cond2(yes)->op1->e
cond2(no)->cond3
cond3(yes)->op2->e
cond3(no)->op3->e
```



##### 3.3.9 Getter和Setter

* ES5中可以使用getter和setter部分改写[[Get]]和[[Put]]的默认操作，但是只能应用在单个属性上，无法应用在整个对象上。getter和setter是隐藏函数。
* 当给属性定义getter、setter或者两者都有时，这个属性会被定义为”**访问描述符**“（和”数据描述符“相对）。会忽略它们的value和writable特性。关心的是set和get(还有configurable和enumerable)特性。

```Js
// 对象文字语法
var myObject = {
    // 给a定义一个getter
    get a() {
        return 2;
    }
};


// defineProperty(..)显式定义
Object.defineProperty(
	myObject,	// 目标对象
    "b", 		// 属性名
    {
        // 给b设置一个getter访问描述符
        get: function() { return this.a * 2 },
        
        // 确保b会出现在对象的属性列表中
        enumerable: true
    }
);

myObject.a; // 2

myObject.b; // 4

myObject.a = 3; // 没有定义setter，set操作无效

myObject.a; // 2
```



* 定义setter

```Js
var myObject = {
    // 给a定义一个getter
    get a() {
        return this._a_; // _a_ 只是一种惯例，没有任何特殊的行为，和其他普通属性一样
    },
    
    // 给a定义一个setter
    set a(val) {
        this._a_ = val * 2;
    }
};

myObject.a = 2;

myObject.a; // 4
```



##### 3.3.10 存在性

* `in`：检查**属性名**是否在对象及其[[Prototype]]原型链中

  * 对数组来说，`4 in [2, 4, 6]`返回false，因为这个数组包含的属性名是0、1、2

  

* `hasOwnProperty(..)`：只会检查属性名是否在对象中

  * 对于`Object.create(null)`对象没有连接到Object.prototype，此时需要`Object.prototype.hasOwnProperty.call(myObject, "a")`显式绑定到myObject。

```Js
var myObject = {
    a: 2
};

("a" in myObject); // true
("b" in myObject); // false

myObject.hasOwnProperty( "a" ); // true
myObject.hasOwnProperty( "b" ); // false
```



* `for..in`循环：遍历对象的可枚举**属性名**列表，包括[[Prototype]]原型链。

  * 对于数组，不仅会包含所有数值索引，还会包含所有可枚举属性。遍历数组使用传统for循环

  

* `propertyIsEnumerable(..)`：只检查属性名是否在对象中并且`enumerable:true`。

* `Object.keys(..)`：只查找属性名是否在对象中，返回一个数组，包含所有可枚举**属性名**。

* `Object.getOwnPropertyNames(..)`：只查找属性名是否在对象中，返回一个数组，包含所有**属性名**，无论是否可枚举。

```Js
var myObject = {};

Object.defineProperty( 
    myObject, 
    "a", 
    // 让a像普通属性一样可以枚举
    { enumerable: true, value: 2 } 
);

Object.defineProperty( 
    myObject, 
    "b", 
    // 让b不可枚举
    { enumerable: false, value: 3} 
);

myObject.b; // 3
("b" in myObject); // true
myObject.hasOwnProperty( "b" ); // true

for(var k in myObject) {
    console.log( k, myObject[k] );
}
// "a" 2

myObject.propertyIsEnumerable( "a" ); // true
myObject.propertyIsEnumerable( "b" ); // false

Object.keys( myObject ); // ["a"]
Object.getOwnPropertyNames( myObject ); // ["a", "b"]
```



#### 3.4 遍历

遍历对象属性时的顺序是不确定的。

* `forEach(..)`遍历数组中的所有值并忽略回调函数的返回值。

* `every(..)`一直运行直到回调函数返回false（或者“假”值），会提前终止遍历

* `some(..)`一直运行直到回调函数返回true（或者“真”值），会提前终止遍历

* `for..of`：直接**遍历值**

  * 数组：直接遍历值而不是数组下标。

    ```Js
    var myArray = [ 1, 2, 3 ];
    
    for(var v of myArray) {
        console.log( v );
    }
    
    // 1
    // 2
    // 3
    ```

    

    数组有内置的`@@iterator`，因此`for..of`可以直接应用在数组上。使用ES6的`Symbol.iterator`来获取对象的`@@iterator`内部属性。`@@iterator`本身并不是一个迭代器对象，而是一个**返回迭代器对象的函数**。

    ```Js
    // 手动遍历数组
    var myArray[ 1, 2, 3 ];
    var it = myArray[Symbol.iterator]();
    
    it.next(); // { value: 1, done: false }
    it.next(); // { value: 2, done: false }
    it.next(); // { value: 3, done: false }
    it.next(); // { done: true }
    ```

    

  * 对象：定义了迭代器就可以遍历。先向被访问对象请求一个迭代器对象，然后通过调用迭代器对象的`next()`方法遍历所有值。

    普通对象没有内置的`@@iterator`，无法自动完成`for..of`遍历。

    ```Js
    // 手动给对象定义@@iterator
    
    var myObject = {
        a: 2,
        b: 3
    };
    
    Object.defineProperty( myObject, Symbol.iterator, {
        enumerable: false,
        writable: false,
        configurable: true,
        value: function() {
            var o = this;
            var idx = 0;
            var ks = Object.keys( o );
            return {
                next: function() {
                    return {
                        value: o[ks[idx++]],
                        done: (idx > ks.length) // 结束条件
                    }
                }
            }
        }
    } );
    
    // 手动遍历myObject
    var it = myArray[Symbol.iterator]();
    it.next(); // { value: 2, done: false }
    it.next(); // { value: 3, done: false }
    it.next(); // { value: undefined, done: true }
    
    // 用for..of遍历myObject
    for (var v of myObject ) {
        console.log( v );
    }
    // 2
    // 3
    ```

    

### 第4章 混合对象“类”

* 类是一种设计模式。许多语言提供了对于面向类软件设计的原生语法。JS也有类似的语法，但是和其他语言中的类完全不同。
* 类意味着复制。
* 传统的类被实例化时，它的行为会被**复制**到实例中。类被继承时，行为也会被**复制**到子类中。
* 多态（在继承链的不同层次名称相同但是功能不同的函数）看起来似乎是从子类引用父类，但是本质上引用的是复制的结果。
* 多态并不表示子类和父类有关联，子类得到的只是父类的一份副本。类的继承其实就是**复制**。
* JS并不会（像类那样）自动创建对象的副本，不会自动执行复制行为。
* 混入模式（无论显式还是隐式）可以用来模拟类的复制行为，但是通常会产生丑陋并且脆弱的语法，比如显式伪多态（`OtherObj.methodName.call(this, ...)`）,会让代码更加难懂并且难以维护。
* 显式混入实际上无法完全模拟类的复制行为，因为对象（函数也是对象）只能复制引用，无法复制被引用的对象或者函数本身。
* 在JS中模拟类是得不偿失的。



#### 4.1 混入

* 混入用来模拟类的复制行为。

  

##### 4.1.1 显式混入

* 混入处理的已经不再是类了，因为在JS中不存在类，`Vehicle`和`Car`都是对象，进行复制和粘贴。

```Js
// 非常简单的mixin(..)例子：
function mixin( sourceObj, targetObj ) {
    for(var key in sourceObj) {
        // 只会在不存在的情况下复制
        if(!(key in targetObj)) {
            targetObj[key] = sourceObj[key];
        }
    }
    return targetObj;
}

var Vehicle = {
    engines: 1,
    
    ignition: function() {
        console.log( "Turning on my engine." );
    },
    
    drive: function() {
        this.ignition();
        console.log( "Steering and moving forward!" );
    }
};

var Car = mixin( Vehicle, {
    wheels: 4,
    
    drive: function() {
        Vehicle.drive.call( this ); // 显示多态(伪多态)调用，因为Vehicle和Car中都有drive
        console.log(
        	"Rolling on all" + this.wheels + "wheels!"
        );
    }
} );
```



* 另一种混入模式，先复制然后对Car进行特殊化，可以跳过存在性检查。不如第一种方法常用。

```Js
// 可能有重写风险
function mixin( sourceObj, targetObj ) {
    for(var key in sourceObj) {
        targetObj[key] = sourceObj[key];
    }
    return targetObj;
}

var Vehicle = {
    // ...
};

// 首先创建一个空对象并把Vehicle的内容复制进去
var Car = mixin( Vehicle, { });

// 然后把新内容复制到Car中
mixin( {
    wheels: 4,
    
    drive: function() {
        // ...
    }
}, Car );
```



* 寄生继承，既是显式的又是隐式的

```Js
// “传统的JS类” Vehicle
function Vehicle() {
    this.engines = 1;
}
Vehicle.prototype.ignition = function() {
    console.log( "Turning on my engine." );
};
Vehicle.prototype.drive = function() {
    this.ignition();
    console.log( "Steering and moving forward!" );
}

// "寄生类"Car
function Car() {
    // 首先，car是一个Vehicle
    var car = new Vehicle();
    
    // 接着我们对car进行定制
    car.wheels = 4;
    
    // 保存到Vehicle::drive()的特殊引用
    var vehDrive = car.drive();
    
    // 重写Vehicle::drive()
    car.drive = function() {
        vehDrive.call( this );
        console.log(
        	"Rolling on all " + this.wheels + " wheels!"
        );
    }
    return car;
}

var myCar = new Car();
myCar.drive();
// Turning on my engine.
// Steering and moving forward!
// Rolling on all 4 wheels!
```

调用`new Car()`时会创建一个新对象并绑定到Car的this上，上述代码没有使用这个新对象而是返回了我们自己的car对象，最初创建的这个对象会被丢弃。因此可以不使用new关键字调用Car()。结果是一致的，但是可以避免创建并丢弃多余的对象。



##### 4.1.2 隐式混入

```Js
var Something = {
    cool: function() {
        this.greeting = "Hello World";
        this.count = this.count ? this.count + 1 : 1;
    }
};

Something.cool();
Something.greeting; // "Hello World"
Something.count; // 1

var Another = {
    cool: function() {
        // 隐式把Something混入Another
        Something.cool.call( this );
    }
};

Another.cool();
Another.greeting; // "Hello World"
Another.count; // 1 (count不是共享状态)
```



### 第5章 原型

#### 5.1 [[Prototype]]

##### 5.1.1 Object.prototype

##### 5.1.2 属性设置和屏蔽



#### 5.2 "类"

##### 5.2.1 “类”函数

##### 5.2.2 “构造函数”

##### 5.2.3 技术



#### 5.3 （原型）继承



#### 5.4 对象关联

##### 5.4.1 创建关联

##### 5.4.2 关联关系是备用



#### 5.5 小结



### 第6章 行为委托