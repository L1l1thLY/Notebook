#  JavaScript this详解

英文原文地址：[All this](http://bjorn.tipling.com/all-this)

## 全局作用域下的this

在浏览器中，当`this`处于全局作用域时，`this`和`window`是相同的。  

```
<script type="text/javascript">
console.log(this === window); //true
</script>
```

在浏览器中，使用`var`在全局作用域下定义变量等价于为`this`或者`window`添加属性。  

```
<script type="text/javascript">
    var foo = "bar";
    console.log(this.foo); //logs "bar"
    console.log(window.foo); //logs "bar"
</script>
```

如果声明变量时（即使未在全局作用域下，比如函数中）没有使用`var`或`let`（ES6），你实际上是添加或改变一个全局作用域下`this`的属性。

```
<script type="text/javascript">
    foo = "bar";

    function testThis() {
      foo = "foo";
    }

    console.log(this.foo); //logs "bar"
    testThis();
    console.log(this.foo); //logs "foo"
</script>
```

如果你使用了node的repl环境，此时`this`是最顶层的命名空间。`global`也是指向它的。

```
> this
{ ArrayBuffer: [Function: ArrayBuffer],
  Int8Array: { [Function: Int8Array] BYTES_PER_ELEMENT: 1 },
  Uint8Array: { [Function: Uint8Array] BYTES_PER_ELEMENT: 1 },
  ...
> global === this
true
```

但在直接执行一个脚本文件的时候，全局作用域下的`this`一开始是一个空对象。和`global`并不同。

```
//test.js
console.log(this);
console.log(this === global);
```

```
$ node test.js
{}
false
```

但如果你使用repl环境来做同样的事情，结果就不同了。

```
> var foo = "bar";
> this.foo
bar
> global.foo
bar
```

使用node直接执行脚本文件，创建变量不加`var`和`let`会把这个变量添加给`global`，但不会添加给处于脚本文件最顶层作用域的`this`。

```
//test.js
foo = "bar";
console.log(this.foo);
console.log(global.foo);
```

```
$ node test.js
undefined
bar
```

但在node的repl环境下，会添加给`this`和`global`两者。

## 函数作用域下的this
除了处于DOM事件处理函数和提供了`thisArg`参数两种情况（请看下面）之外的情况下。如果你在调用函数时没有使用`new`，`this`是等同于全局作用域下this的。

```
<script type="text/javascript">
    foo = "bar";

    function testThis() {
      this.foo = "foo";
    }

    console.log(this.foo); //logs "bar"
    testThis();
    console.log(this.foo); //logs "foo"
</script>
```

```
//test.js
foo = "bar";

function testThis () {
  this.foo = "foo";
}

console.log(global.foo);
testThis();
console.log(global.foo);
```

```
$ node test.js
bar
foo
```

除非你使用了`"use strict";`，在这种情况下（函数作用域下的）`this`将会是`undefined`。

```
<script type="text/javascript">
    foo = "bar";

    function testThis() {
      "use strict";
      this.foo = "foo";
    }

    console.log(this.foo); //打印 "bar"
    testThis();  //Uncaught TypeError: Cannot set property 'foo' of undefined 
</script>
```

如果你使用`new`调用函数，`this`则指向了一个新环境，不在指向全局作用域下的`this`。（应该是指不再和全局作用域下的this指向相同）

```
<script type="text/javascript">
    foo = "bar";

    function testThis() {
      this.foo = "foo";
    }

    console.log(this.foo); //logs "bar"
    new testThis();
    console.log(this.foo); //logs "bar"

    console.log(new testThis().foo); //logs "foo"
</script>
```

我倾向于叫这个新环境为实例。

## 原型对象中的this
被创建的函数实际上是一个个函数对象。它们都会拥有一个特殊的`prototype`属性，你可以给这个属性赋值。通过`new`调用函数来创建实例，便可以使用被赋予给`prototype`的属性。你将会利用`this`来使用这些值。

```
    function Thing() {
      console.log(this.foo);
    }

    Thing.prototype.foo = "bar";

    var thing = new Thing(); //logs "bar"
    console.log(thing.foo);  //logs "bar"
```

如果使用`new`创建了多个新实例，这些实例会共享在`prototype`上定义的值。举个例子，如果你没有在每个实例中单独重载`this.foo`，当你使用`this.foo`将会会返回同一个值。

```
function Thing() {
}
Thing.prototype.foo = "bar";
Thing.prototype.logFoo = function () {
    console.log(this.foo);
}
Thing.prototype.setFoo = function (newFoo) {
    this.foo = newFoo;
}

var thing1 = new Thing();
var thing2 = new Thing();

thing1.logFoo(); //logs "bar"
thing2.logFoo(); //logs "bar"

thing1.setFoo("foo");
thing1.logFoo(); //logs "foo";
thing2.logFoo(); //logs "bar";

thing2.foo = "foobar";
thing1.logFoo(); //logs "foo";
thing2.logFoo(); //logs "foobar";
```

在实例中的`this`亦是一种特殊的对象，`this`实际上是一个关键字。你可以认为`this`是使用定义在`prototype`上的值的渠道，但是在实例中直接对`this`赋值的行为却会隐藏本来定义在`prototype`上的值。当然如果你做了这样的事，想要再次获得本来定义在`prototype`上的值，可以删除你在实例中对`this`赋的值……

```
function Thing() {
}
Thing.prototype.foo = "bar";
Thing.prototype.logFoo = function () {
    console.log(this.foo);
}
Thing.prototype.setFoo = function (newFoo) {
    this.foo = newFoo;
}
Thing.prototype.deleteFoo = function () {
    delete this.foo;
}

var thing = new Thing();
thing.setFoo("foo");
thing.logFoo(); //logs "foo";
thing.deleteFoo();
thing.logFoo(); //logs "bar";
thing.foo = "foobar";
thing.logFoo(); //logs "foobar";
delete thing.foo;
thing.logFoo(); //logs "bar";
```

……或直接引用函数对象的`prototype`。

```
function Thing() {
}
Thing.prototype.foo = "bar";
Thing.prototype.logFoo = function () {
    console.log(this.foo, Thing.prototype.foo);
}

var thing = new Thing();
thing.foo = "foo";
thing.logFoo(); //logs "foo bar";
```

同一个函数创建的多个实例会共享`prototype`上的值。如果你赋值一个数组给`prototype`，多个实例会共享这个数组，除非你在实例中重写了它，也就是说你隐藏了它。

```
function Thing() {
}
Thing.prototype.things = [];


var thing1 = new Thing();
var thing2 = new Thing();
thing1.things.push("foo");
console.log(thing2.things); //logs ["foo"]
```

一般来说，赋予`prototype`数组或对象都是错误的行为。如果你需要每个实例拥有他们自己的数组，那就在函数里创造，而不是在`prototype`上。

```
function Thing() {
    this.things = [];
}


var thing1 = new Thing();
var thing2 = new Thing();
thing1.things.push("foo");
console.log(thing1.things); //logs ["foo"]
console.log(thing2.things); //logs []
```

将多个函数的`prototype`连成一条原型链（prototype chain），这样`this`就会魔法般地沿着原型链向上，直到找到你需要的值。

```
function Thing1() {
}
Thing1.prototype.foo = "bar";

function Thing2() {
}
Thing2.prototype = new Thing1();


var thing = new Thing2();
console.log(thing.foo); //logs "bar"
```

一些人会利用这个在JS中模拟经典面对对象的继承。  

在已经存在于原型链的函数中，对于`this`所有的赋值都会隐藏沿着原型链链往上定义的同名值。

```
function Thing1() {
}
Thing1.prototype.foo = "bar";

function Thing2() {
    this.foo = "foo";
}
Thing2.prototype = new Thing1();

function Thing3() {
}
Thing3.prototype = new Thing2();


var thing = new Thing3();
console.log(thing.foo); //logs "foo"
```

我喜欢叫赋给`prototype`的值为「方法」。我已经使用了一些方法在上面的例子里，比如`logFoo`。这些方法都会获得一个「有魔力」的`this`原型，如同最开始那个用来创建实例的函数一样。我一般会叫这个原始函数为构造函数。  

在`prototype`上定义的方法里使用的`this`，无论在继承链的任何位置，都引用着当前的实例。这也就意味着，如果通过在继承链上对`this`直接复制来隐藏一个值，此实例上的方法都会使用新值，而不会再考虑这个方法实际是赋给哪个`prototype`的。

```
function Thing1() {
}
Thing1.prototype.foo = "bar";
Thing1.prototype.logFoo = function () {
    console.log(this.foo);
}

function Thing2() {
    this.foo = "foo";
}
Thing2.prototype = new Thing1();


var thing = new Thing2();
thing.logFoo(); //logs "foo";
```
    
在JavaScript中你可以嵌套定义函数，也就是说你可以在一个函数中定义函数。然而嵌套在内部的函数在一个闭包中捕获其外部函数的变量，并不会继承`this`。

```
function Thing() {
}
Thing.prototype.foo = "bar";
Thing.prototype.logFoo = function () {
    var info = "attempting to log this.foo:";
    function doIt() {
        console.log(info, this.foo);
    }
    doIt();
}


var thing = new Thing();
thing.logFoo();  //logs "attempting to log this.foo: undefined"
```

在`"use strict"`情况下，在`doIt`中的`this`是`undefined`，而默认情况下是`global`对象。这对于不熟悉JS的`this`来说，也是一项痛苦的来源。  

更糟糕的是，把一个实例方法当做值，比如把一个方法当做参数传给一个函数，并不会把其所在实例一起传给函数。当这种情况发生时，`this`会再次指向`global`对象，或者在`"use strict";`情况下的的`undefined`。

```
function Thing() {
}
Thing.prototype.foo = "bar";
Thing.prototype.logFoo = function () {  
    console.log(this.foo);   
}

function doIt(method) {
    method();
}


var thing = new Thing();
thing.logFoo(); //logs "bar"
doIt(thing.logFoo); //logs undefined
```

有些人更倾向于在变量中捕获`this`，一般会把用于捕获的变量起名为「self」，避开了`this`.……

```
function Thing() {
}
Thing.prototype.foo = "bar";
Thing.prototype.logFoo = function () {
    var self = this;
    var info = "attempting to log this.foo:";
    function doIt() {
        console.log(info, self.foo);
    }
    doIt();
}


var thing = new Thing();
thing.logFoo();  //logs "attempting to log this.foo: bar"
```

……但是这仍然不会解决把方法当成参数传递不会传递实例的问题。

```
function Thing() {
}
Thing.prototype.foo = "bar";
Thing.prototype.logFoo = function () { 
    var self = this;
    function doIt() {
        console.log(self.foo);
    }
    doIt();
}

function doItIndirectly(method) {
    method();
}


var thing = new Thing();
thing.logFoo(); //logs "bar"
doItIndirectly(thing.logFoo); //logs undefined
```

对于所有的定义在函数对象上的函数和方法，通过`bind`，便可以连实例带方法一起传递，

```
function Thing() {
}
Thing.prototype.foo = "bar";
Thing.prototype.logFoo = function () { 
    console.log(this.foo);
}

function doIt(method) {
    method();
}


var thing = new Thing();
doIt(thing.logFoo.bind(thing)); //logs bar
```

你也可以使用`apply`和`call`，在一个新环境中调用方法或函数。

```
function Thing() {
}
Thing.prototype.foo = "bar";
Thing.prototype.logFoo = function () { 
    function doIt() {
        console.log(this.foo);
    }
    doIt.apply(this);
}

function doItIndirectly(method) {
    method();
}


var thing = new Thing();
doItIndirectly(thing.logFoo.bind(thing)); //logs bar
```
> 译者注：这里的代码同时出现了两个内容点，有点难以理解，稍微解释一下，`apply``call``bind`都是用来改变一个方法调用环境的方法，前两者用处几乎相同，为直接改变函数的调用环境并调用，而`bind`则是只返回改变了环境的函数。上面的这个代码例子，可见我们在Thing构造函数中定义了`logFoo`方法，而此方法其中有一个局部函数，由上文可知，此局部函数中的`this`是指向着`global`或`undefined`的。然后，通过`doIt.apply(this);`改变了局部函数中的`this`指向，从而使其变为了原型对象`this`，并执行。而下文中的`doItIndirectly(thing.logFoo.bind(thing)); `，如果没有`bind`语句，由于上文提到「把一个实例方法当做值，比如把一个方法当做参数传给一个函数，并不会把其所在实例一起传给函数」，故使用`bind`改变了`log.Foo`的执行环境为实例`this`再传给了`doItIndirectly`。

你可以利用`bind`来替换一个函数或方法的`this`指向，即使它本来并没有赋予给这个实例的原型。

```
function Thing() {
}
Thing.prototype.foo = "bar";


function logFoo(aStr) {
    console.log(aStr, this.foo);
}


var thing = new Thing();
logFoo.bind(thing)("using bind"); //logs "using bind bar"
logFoo.apply(thing, ["using apply"]); //logs "using apply bar"
logFoo.call(thing, "using call"); //logs "using call bar"
logFoo("using nothing"); //logs "using nothing undefined"
```

你应该避免你的构造函数返回任何值，因为这会改变结果对象的指向。

```
function Thing() {
    return {};
}
Thing.prototype.foo = "bar";


Thing.prototype.logFoo = function () {
    console.log(this.foo);
}


var thing = new Thing();
thing.logFoo(); //Uncaught TypeError: undefined is not a function
```

奇怪的是，如果构造函数返回一个基本数据类型值，比如一个字符串或一个数值，这种情况就不会发生，同时返回语句也会被忽略。最好的方式是永远不要在一个你可能会用`new`调用的函数中返回任何值。如果你想要使用工厂模式，使用函数创造实例，并不使用`new`，这个看法就仅供参考了。  


你可以避免使用`new`而是使用`Object.create`。这也会创造一个实例。

```
function Thing() {
}
Thing.prototype.foo = "bar";


Thing.prototype.logFoo = function () {
    console.log(this.foo);
}


var thing =  Object.create(Thing.prototype);
thing.logFoo(); //logs "bar"
```

这种方式不会调用构造函数。

```
function Thing() {
    this.foo = "foo";
}
Thing.prototype.foo = "bar";


Thing.prototype.logFoo = function () {
    console.log(this.foo);
}


var thing =  Object.create(Thing.prototype);
thing.logFoo(); //logs "bar"
```

因为`Obeject.create`不会调用构造函数，所以当你使用继承模式，准备在继承链更下层处重写构造函数时，就很有用了。

```
function Thing1() {
    this.foo = "foo";
}
Thing1.prototype.foo = "bar";

function Thing2() {
    this.logFoo(); //logs "bar"
    Thing1.apply(this);
    this.logFoo(); //logs "foo"
}
Thing2.prototype = Object.create(Thing1.prototype);
Thing2.prototype.logFoo = function () {
    console.log(this.foo);
}

var thing = new Thing2();
```

##对象中的this
可以在一个对象中的函数中使用`this`来引用此对象上的其他属性。这和使用new来创建一个实例是不同的。

```
var obj = {
    foo: "bar",
    logFoo: function () {
        console.log(this.foo);
    }
};

obj.logFoo(); //logs "bar"
```

注意，`obj`不是通过`new`也不是通过`Object.create`创建的。你也可以把函数绑定给这种对象，就如同这些对象也是实例一样。

```
var obj = {
    foo: "bar"
};

function logFoo() {
    console.log(this.foo);
}

logFoo.apply(obj); //logs "bar"
```

当你像下面这样使用`this`的时候，它不会向下深入挖掘继承层次。只有相邻层次的父母对象所拥有的属性（和函数同层次）才能通过`this`访问。

```
var obj = {
    foo: "bar",
    deeper: {
        logFoo: function () {
            console.log(this.foo);
        }
    }
};

obj.deeper.logFoo(); //logs undefined
```

你只能直接引用属性:

```
var obj = {
    foo: "bar",
    deeper: {
        logFoo: function () {
            console.log(obj.foo);
        }
    }
};

obj.deeper.logFoo(); //logs "bar"
```

##DOM事件中的属性

在一个HTML DOM事件处理函数中，`this`永远指向事件所对应的DOM元素……

```
function Listener() {
    document.getElementById("foo").addEventListener("click",
       this.handleClick);
}
Listener.prototype.handleClick = function (event) {
    console.log(this); //logs "<div id="foo"></div>"
}

var listener = new Listener();
document.getElementById("foo").click();
```

……除非你`bind`了环境。

```
function Listener() {
    document.getElementById("foo").addEventListener("click", 
        this.handleClick.bind(this));
}
Listener.prototype.handleClick = function (event) {
    console.log(this); //logs Listener {handleClick: function}
}

var listener = new Listener();
document.getElementById("foo").click();
```

##HTML相关的this
在HTML特性中你可以存放一些JS，`this`指向所在元素。

```
<div id="foo" onclick="console.log(this);"></div>
<script type="text/javascript">
document.getElementById("foo").click(); //logs <div id="foo"...
</script>
```
##重写this
你不能重写`this`因为它是一个关键字。

```
function test () {
    var this = {};  // Uncaught SyntaxError: Unexpected token this 
}
```

##eval this
你可以使用`eval`访问`this`。

```
function Thing () {
}
Thing.prototype.foo = "bar";
Thing.prototype.logFoo = function () {
    eval("console.log(this.foo)"); //logs "bar"
}

var thing = new Thing();
thing.logFoo();
```

这将会成为一个安全隐患。除非不适用`eval`不然没办法避免。  

使用`Function`来创建函数同样可以访问`this`：

```
function Thing () {
}
Thing.prototype.foo = "bar";
Thing.prototype.logFoo = new Function("console.log(this.foo);");

var thing = new Thing();
thing.logFoo(); //logs "bar"
```

##with this
你可以使用`with`来将`this`引入当前环境使用而不需要实际引用`this`。

```
function Thing () {
}
Thing.prototype.foo = "bar";
Thing.prototype.logFoo = function () {
    with (this) {
        console.log(foo);
        foo = "foo";
    }
}

var thing = new Thing();
thing.logFoo(); // logs "bar"
console.log(thing.foo); // logs "foo"
```

很多人认为这种写法很糟糕，因为会引起`with`相关的歧义问题。

##jQuery this
如同对于HTML DOM的事件处理函数一样，jq库很多地方使用`this`来引用DOM元素。对于事件处理函数和一些方便的方法（比如`$.each`），`this`引用DOM元素。  

```
<div class="foo bar1"></div>
<div class="foo bar2"></div>
<script type="text/javascript">
$(".foo").each(function () {
    console.log(this); //logs <div class="foo...
});
$(".foo").on("click", function () {
    console.log(this); //logs <div class="foo...
});
$(".foo").each(function () {
    this.click();
});
</script>
```

##thisArg this
如果你使用 *underscore.js*  或 *lo-dash* 你会知道，在库中，很多方法可以传入一个实例，通过`thisArg`函数参数，这个参数将会成为`this`环境。举个例子，`_.each`就是这样的。从ES5开始，原生的方法就允许`thisArg`，比如`forEach`。实际上，之前的`bind`，`apply`和`call`的实例用法就教你一些使用`thisArg`的途径。实际上当你使用`bind`时，是将对象作为`thisArg`传入了。

```
function Thing(type) {
    this.type = type;
}
Thing.prototype.log = function (thing) {
    console.log(this.type, thing);
}
Thing.prototype.logThings = function (arr) {
   arr.forEach(this.log, this); // logs "fruit apples..."
   _.each(arr, this.log, this); //logs "fruit apples..."
}

var thing = new Thing("fruit");
thing.logThings(["apples", "oranges", "strawberries", "bananas"]);
```

不用再繁琐地使用bind语句，也不用采用self代替this，代码显然更加清晰简洁了。


