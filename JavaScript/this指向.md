# this指向

### 1.关于this对象

this对象是在**运行时**基于函数的执行环境绑定的（this 实际上是在函数被调用时发生的绑定，它指向什么完全取决于函数在哪里被调用，也就是函数的调用位置）。

在全局函数中，this等于window;

当函数被作为某个对象的方法调用时，this等于那个对象；

匿名函数的执行环境具有全局性，因此其this对象通常指向window;

```
var name = "the window";
var object = {
	name: "my object",
	getNameFunc: function(){
		return this.name;
	}
};

alert(object.getNameFunc()()); //"the window"(在非严格模式下)
```

### 2.函数的调用位置

通常来说，寻找调用位置就是寻找“函数被调用的位置”，但是做起来并没有这么简单，因为某些编程模式可能会隐藏真正的调用位置。

最重要的是要分析调用栈（就是为了到达当前执行位置所调用的所有函数）。我们关心的
调用位置就在**当前正在执行的函数的前一个调用中**。

下面我们来看看到底什么是调用栈和调用位置：

```
function baz() {
    // 当前调用栈是：baz
    // 因此，当前调用位置是全局作用域
    console.log( "baz" );
    bar(); // <-- bar 的调用位置
}
function bar() {
    // 当前调用栈是 baz -> bar
    // 因此，当前调用位置在 baz 中
    console.log( "bar" );
    foo(); // <-- foo 的调用位置
}
function foo() {
    // 当前调用栈是 baz -> bar -> foo
    // 因此，当前调用位置在 bar 中
    console.log( "foo" );
}
baz(); // <-- baz 的调用位置
```

注意我们是如何（从调用栈中）分析出真正的调用位置的，因为它决定了 this 的绑定.

### 3、this的绑定规则

我们来看看在函数的执行过程中调用位置如何决定 this 的绑定对象。

你必须找到调用位置，然后判断需要应用下面四条规则中的哪一条。我们首先会分别解释
这四条规则，然后解释多条规则都可用时它们的优先级如何排列。

##### 1）默认绑定（普通函数调用）

```
function foo() {
    console.log( this.a );
}
var a = 2;
foo(); // 2
```

我们可以看到当调用 foo() 时，this.a 被解析成了全局变量 a。为什么？因为在本例中，函数调用时应用了 this 的默认绑定，因此 this 指向全局对象。

那么我们怎么知道这里应用了默认绑定呢？可以通过分析调用位置来看看 foo() 是如何调用的。在代码中，foo() 是直接使用不带任何修饰的函数引用进行调用的，因此只能使用默认绑定，无法应用其他规则。

如果使用严格模式（strict mode），那么全局对象将无法使用默认绑定，因此 this 会绑定到 undefined：

```
function foo() {
    "use strict";
    console.log( this.a );
}
var a = 2;
foo(); // TypeError: this is undefined
```

这里有一个微妙但是非常重要的细节，虽然 this 的绑定规则完全取决于调用位置，但是只有 foo() 运行在非 strict mode 下时，默认绑定才能绑定到全局对象；严格模式下与 foo()的调用位置无关：

```
function foo() {
    console.log( this.a );
}
var a = 2;
(function(){
    "use strict";
    foo(); // 2
})();
```

##### 2）隐式绑定（对象函数调用）

思考下面的代码：

```
function foo() {
    console.log( this.a );
}
var obj = {
    a: 2,
    foo: foo
};
obj.foo(); // 2
```

首先需要注意的是 foo() 的声明方式，及其之后是如何被当作引用属性添加到 obj 中的。但是无论是直接在 obj 中定义还是先定义再添加为引用属性，这个函数严格来说都不属于obj 对象。

然而，调用位置会使用 obj 上下文来引用函数，因此你可以说函数被调用时 obj 对象“拥有”或者“包含”它。

无论你如何称呼这个模式，当 foo() 被调用时，它的落脚点确实指向 obj 对象。<u>当函数引用有上下文对象时，隐式绑定规则会把函数调用中的 this 绑定到这个上下文对象</u>。因为调用 foo() 时 this 被绑定到 obj，因此 this.a 和 obj.a 是一样的。

对象属性引用链中只有最顶层或者说最后一层会影响调用位置。举例来说：

```
function foo() {
    console.log( this.a );
}
var obj2 = {
    a: 42,
    foo: foo
};
var obj1 = {
    a: 2,
    obj2: obj2
};
obj1.obj2.foo(); // 42
```

**隐式丢失：**

一个最常见的 this 绑定问题就是被隐式绑定的函数会丢失绑定对象，也就是说它会应用默认绑定，从而把 this 绑定到全局对象或者 undefined 上，取决于是否是严格模式。

思考下面的代码：

```
function foo() {
    console.log( this.a );
}
var obj = {
    a: 2,
    foo: foo
};
var bar = obj.foo; // 函数别名！
var a = "oops, global"; // a 是全局对象的属性
bar(); // "oops, global"
```

虽然 bar 是 obj.foo 的一个引用，但是实际上，它引用的是 foo 函数本身，因此此时的bar() 其实是一个不带任何修饰的函数调用，因此应用了默认绑定。

一种更微妙、更常见并且更出乎意料的情况发生在传入回调函数时：

```
function foo() {
    console.log( this.a );
}
function doFoo(fn) {
    // fn 其实引用的是 foo
    fn(); // <-- 调用位置！
}
var obj = {
    a: 2,
    foo: foo
};
var a = "oops, global"; // a 是全局对象的属性
doFoo( obj.foo ); // "oops, global"
```

参数传递其实就是一种隐式赋值，因此我们传入函数时也会被隐式赋值，所以结果和上一个例子一样。

如果把函数传入语言内置的函数而不是传入你自己声明的函数，会发生什么呢？结果是一样的，没有区别：

```
function foo() {
    console.log( this.a );
}
var obj = {
    a: 2,
    foo: foo
};
var a = "oops, global"; // a 是全局对象的属性
setTimeout( obj.foo, 100 ); // "oops, global"

setTimeout(foo.call(obj), 100); //"2"
```

就像我们看到的那样，回调函数丢失 this 绑定是非常常见的。除此之外，还有一种情况 this 的行为会出乎我们意料：调用回调函数的函数可能会修改 this。<u>在一些流行的JavaScript 库中事件处理器常会把回调函数的 this 强制绑定到触发事件的 DOM 元素上</u>。这在一些情况下可能很有用，但是有时它可能会让你感到非常郁闷。遗憾的是，这些工具通常无法选择是否启用这个行为。

无论是哪种情况，this 的改变都是意想不到的，实际上你无法控制回调函数的执行方式，因此就没有办法控制会影响绑定的调用位置。之后我们会介绍如何通过固定 this 来修复（这里是双关，“修复”和“固定”的英语单词都是 fixing）这个问题。

##### 3）显式绑定

就像我们刚才看到的那样，在分析隐式绑定时，我们必须在一个对象内部包含一个指向函数的属性，并通过这个属性间接引用函数，从而把 this 间接（隐式）绑定到这个对象上。

那么如果我们不想在对象内部包含函数引用，而想在某个对象上强制调用函数，该怎么做呢？

JavaScript 中的“所有”函数都有一些有用的特性（这和它们的 [[ 原型 ]] 有关），可以用来解决这个问题。具体点说，可以使用函数的 call(..) 和apply(..) 方法。严格来说，JavaScript 的宿主环境有时会提供一些非常特殊的函数，它们并没有这两个方法。但是这样的函数非常罕见，JavaScript 提供的绝大多数函数以及你自己创建的所有函数都可以使用 call(..) 和 apply(..) 方法。这两个方法是如何工作的呢？

它们的第一个参数是一个对象，它们会把这个对象绑定到this，接着在调用函数时指定这个 this。因为你可以直接指定 this 的绑定对象，因此我们称之为显式绑定。

思考下面的代码：

```
function foo() {
    console.log( this.a );
}
var obj = {
    a:2	
};
foo.call( obj ); // 2
```

通过 foo.call(..)，我们可以在调用 foo 时强制把它的 this 绑定到 obj 上。如果你传入了一个原始值（字符串类型、布尔类型或者数字类型）来当作 this 的绑定对象，这个原始值会被转换成它的对象形式（也就是 new String(..)、new Boolean(..) 或者new Number(..)）。这通常被称为<u>“装箱”</u>。

可惜，显式绑定仍然无法解决我们之前提出的丢失绑定问题。(??)

###### 3.1硬绑定

但是显式绑定的一个变种可以解决这个问题。
思考下面的代码：

```
function foo() {
    console.log( this.a );
}
var obj = {
    a:2s
};
var bar = function() {
    foo.call( obj );
};
bar(); // 2
setTimeout( bar, 100 ); // 2
// 硬绑定的 bar 不可能再修改它的 this
bar.call( window ); // 2
```

我们来看看这个变种到底是怎样工作的。我们创建了函数 bar()，并在它的内部手动调用了 foo.call(obj)，因此强制把 foo 的 this 绑定到了 obj。无论之后如何调用函数 bar，它总会手动在 obj 上调用 foo。这种绑定是一种显式的强制绑定，因此我们称之为硬绑定。

硬绑定的典型应用场景就是创建一个包裹函数，传入所有的参数并返回接收到的所有值：

```
function foo(something) {
    console.log( this.a, something );
    return this.a + something;
}
var obj = {
    a:2
};
var bar = function() {
    return foo.apply( obj, arguments );
};
var b = bar( 3 ); // 2 3
console.log( b ); // 5
```

另一种使用方法是创建一个 i 可以重复使用的辅助函数：

```
function foo(something) {
    console.log( this.a, something );
    return this.a + something;
}
// 简单的辅助绑定函数
function bind(fn, obj) {
    return function() {
    	return fn.apply( obj, arguments );
	};
}
var obj = {
    a:2
};
var bar = bind( foo, obj );
var b = bar( 3 ); // 2 3
console.log( b ); // 5
```

由于硬绑定是一种非常常用的模式，所以在 ES5 中提供了内置的方法 Function.prototype.bind，它的用法如下：

```
function foo(something) {
    console.log( this.a, something );
    return this.a + something;
}
var obj = {
    a:2
};
var bar = foo.bind( obj );
var b = bar( 3 ); // 2 3
console.log( b ); // 5
```

bind(..) 会返回一个硬编码的新函数，它会把参数设置为 this 的上下文并调用原始函数。

##### 4）new绑定

这是第四条也是最后一条 this 的绑定规则，在讲解它之前我们首先需要澄清一个非常常见的关于 JavaScript 中函数和对象的误解。

在传统的面向类的语言中，“构造函数”是类中的一些特殊方法，使用 new 初始化类时会调用类中的构造函数。通常的形式是这样的：

something = new MyClass(..);

JavaScript 也有一个 new 操作符，使用方法看起来也和那些面向类的语言一样，绝大多数开发者都认为JavaScript 中 new 的机制也和那些语言一样。然而，JavaScript 中 new 的机制实际上和面向类的语言完全不同。

使用 new 来调用函数，或者说发生构造函数调用时，会自动执行下面的操作。

1. 创建（或者说构造）一个全新的对象。
2. 这个新对象会被执行 [[ 原型 ]] 连接。
3. 这个新对象会绑定到函数调用的 this。
4. 如果函数没有返回其他对象，那么 new 表达式中的函数调用会自动返回这个新对象。

思考下面的代码：

```
function foo(a) {
    this.a = a;
}
var bar = new foo(2);
console.log( bar.a ); // 2
```

使用 new 来调用 foo(..) 时，我们会构造一个新对象并把它绑定到 foo(..) 调用中的 this上。new 是最后一种可以影响函数调用时 this 绑定行为的方法，我们称之为 new 绑定。

### 4.this的绑定优先级

##### new绑定>显式绑定>隐式绑定>默认绑定

例子如下：

##### 1）隐式绑定与显式绑定对比

```
function foo() {
    console.log( this.a );
}
var obj1 = {
    a: 2,
    foo: foo
};
var obj2 = {
    a: 3,
    foo: foo
};
obj1.foo(); // 2
obj2.foo(); // 3
obj1.foo.call( obj2 ); // 3
obj2.foo.call( obj1 ); // 2
```

可以看到，显式绑定优先级更高，也就是说在判断时应当先考虑是否可以应用显式绑定。

##### 2）new 绑定和隐式绑定的对比

```
function foo(something) {
    this.a = something;
}
var obj1 = {
    foo: foo
};
var obj2 = {};
obj1.foo( 2 );
console.log( obj1.a ); // 2
obj1.foo.call( obj2, 3 );
console.log( obj2.a ); // 3
var bar = new obj1.foo( 4 );
console.log( obj1.a ); // 2
console.log( bar.a ); // 4
```

可以看到 new 绑定比隐式绑定优先级高。

##### 3）new绑定和显式绑定的对比

new 和 call/apply 无法一起使用，因此无法通过 new foo.call(obj1) 来直接进行测试。但是我们可以使用硬绑定来测试它俩的优先级。

在看代码之前先回忆一下硬绑定是如何工作的。Function.prototype.bind(..) 会创建一个新的包装函数，这个函数会忽略它当前的 this 绑定（无论绑定的对象是什么），并把我们提供的对象绑定到 this 上。

这样看起来硬绑定（也是显式绑定的一种）似乎比 new 绑定的优先级更高，无法使用 new来控制 this 绑定。

我们看看是不是这样：

```
function foo(something) {
    this.a = something;
}
var obj1 = {};
var bar = foo.bind( obj1 );
bar( 2 );
console.log( obj1.a ); // 2
var baz = new bar(3);
console.log( obj1.a ); // 2
console.log( baz.a ); // 3
```

出乎意料！ bar 被硬绑定到 obj1 上，但是 new bar(3) 并没有像我们预计的那样把 obj1.a修改为 3。相反，new 修改了硬绑定（到 obj1 的）调用 bar(..) 中的 this。因为使用了new 绑定，我们得到了一个名字为 baz 的新对象，并且 baz.a 的值是 3。

再来看看我们之前介绍的“裸”辅助函数 bind：

```
function bind(fn, obj) {
    return function() {
        fn.apply( obj, arguments );
    };
}
```

非常令人惊讶，因为看起来在辅助函数中 new 操作符的调用无法修改 this 绑定，但是在刚才的代码中 new 确实修改了 this 绑定。

实际上，ES5 中内置的 Function.prototype.bind(..) 更加复杂。下面是 MDN 提供的一种bind(..) 实现，为了方便阅读我们对代码进行了排版：

```
if (!Function.prototype.bind) {
    Function.prototype.bind = function(oThis) {
        if (typeof this !== "function") {
            // 与 ECMAScript 5 最接近的
            // 内部 IsCallable 函数
            throw new TypeError(
            "Function.prototype.bind - what is trying " +
            "to be bound is not callable"
            );
        }
        var aArgs = Array.prototype.slice.call( arguments, 1 ),
        fToBind = this,
        fNOP = function(){},
        fBound = function(){
            return fToBind.apply(
                (
                    this instanceof fNOP &&
                    oThis ? this : oThis
                ),
            aArgs.concat(
            	Array.prototype.slice.call( arguments )
            );
        };
        fNOP.prototype = this.prototype;
        fBound.prototype = new fNOP();
        return fBound;
    };
}
```

这种 bind(..) 是一种 polyfill 代码（polyfill 就是我们常说的刮墙用的腻子，polyfill 代码主要用于旧浏览器的兼容，比如说在旧的浏览器中并没有内置 bind 函数，因此可以使用 polyfill 代码在旧浏览器中实现新的功能），对于 new 使用的硬绑定函数来说，这段 polyfill 代码和 ES5 内置的bind(..) 函数并不完全相同（后面会介绍为什么要在 new 中使用硬绑定函数）。由于 polyfill 并不是内置函数，所以无法创建一个不包含 .prototype的函数，因此会具有一些副作用。如果你要在 new 中使用硬绑定函数并且依赖 polyfill 代码的话，一定要非常小心。

下面是 new 修改 this 的相关代码：

```
this instanceof fNOP &&
oThis ? this : oThis
// ... 以及：
fNOP.prototype = this.prototype;
fBound.prototype = new fNOP();
```

我们并不会详细解释这段代码做了什么（这非常复杂并且不在我们的讨论范围之内），不过简单来说，这段代码会判断硬绑定函数是否是被 new 调用，如果是的话就会使用新创建的 this 替换硬绑定的 this。

那么，为什么要在 new 中使用硬绑定函数呢？直接使用普通函数不是更简单吗？之所以要在 new 中使用硬绑定函数，主要目的是预先设置函数的一些参数，这样在使用new 进行初始化时就可以只传入其余的参数。bind(..) 的功能之一就是可以把除了第一个参数（第一个参数用于绑定 this）之外的其他参数都传给下层的函数（这种技术称为“部分应用”，是“柯里化”的一种）。

举例来说：

```
function foo(p1,p2) {
    this.val = p1 + p2;
}
// 之所以使用 null 是因为在本例中我们并不关心硬绑定的 this 是什么
// 反正使用 new 时 this 会被修改
var bar = foo.bind( null, "p1" );
var baz = new bar( "p2" );
baz.val; // p1p2
```

### 5.this绑定例外

在某些场景下 this 的绑定行为会出乎意料，你认为应当应用其他绑定规则时，实际上应用 的可能是默认绑定规则。

##### 1）被忽略的this

如果你把 null 或者 undefined 作为 this 的绑定对象传入 call、apply 或者 bind，**这些值 在调用时会被忽略，实际应用的是默认绑定规则**：

```
function foo() { 
	console.log( this.a );
}
var a = 2; 
foo.call( null ); // 2
```

那么什么情况下你会传入 null 呢？ 

一种非常常见的做法是使用 apply(..) 来“展开”一个数组，并当作参数传入一个函数。 类似地，bind(..) 可以对参数进行柯里化（预先设置一些参数），这种方法有时非常有用：

```
function foo(a,b) { 
	console.log( "a:" + a + ", b:" + b );
}
// 把数组“展开”成参数 
foo.apply( null, [2, 3] ); // a:2, b:3
// 使用 bind(..) 进行柯里化
var bar = foo.bind( null, 2 ); 
bar( 3 ); // a:2,b:3
```

这两种方法都需要传入一个参数当作 this 的绑定对象。如果函数并不关心 this 的话，你 仍然需要传入一个占位值，这时 null 可能是一个不错的选择，就像代码所示的那样。

<u>尽管本书中未提到，但在 ES6 中，可以用 ... 操作符代替 apply(..) 来“展 开”数组，foo(...[1,2]) 和 foo(1,2) 是一样的，这样可以避免不必要的 this 绑定。可惜，在 ES6 中没有柯里化的相关语法，因此还是需要使用 bind(..)。</u>

然而，总是使用 null 来忽略 this 绑定可能产生一些副作用。如果某个函数确实使用了 this（比如第三方库中的一个函数），那默认绑定规则会把 this 绑定到全局对象（在浏览 器中这个对象是 window），这将导致不可预计的后果（比如修改全局对象）。 

显而易见，这种方式可能会导致许多难以分析和追踪的 bug。

所以我们要使用一种更安全的this,一种“更安全”的做法是传入一个特殊的对象，把 this 绑定到这个对象不会对你的程序 产生任何副作用。

如果我们在忽略 this 绑定时总是传入一个 DMZ 对象(它就是一个空的非委托的对象)，那就什么都不用担心了，因为任何 对于 this 的使用都会被限制在这个空对象中，不会对全局对象产生任何影响。

JavaScript 中创建一个空对象最简单的方法都是 Object.create(null)。Object.create(null) 和 {} 很 像， 但 是 并 不 会 创 建 Object. prototype 这个委托，所以它比 {}“更空”：

```
function foo(a,b) { 
	console.log( "a:" + a + ", b:" + b ); 
}
// 我们的 DMZ 空对象 
var ø = Object.create( null ); 
// 把数组展开成参数 
foo.apply( ø, [2, 3] ); // a:2, b:3 

// 使用 bind(..) 进行柯里化
var bar = foo.bind( ø, 2 ); 
bar( 3 ); // a:2, b:3
```

使用变量名 ø 不仅让函数变得更加“安全”，而且可以提高代码的可读性，因为 ø 表示 “我希望 this 是空”，这比 null 的含义更清楚。但是你也可以任意名命。

##### 2）间接引用

另一个需要注意的是，你有可能（有意或者无意地）创建一个函数的“间接引用”，在这 种情况下，调用这个函数会应用默认绑定规则。 

间接引用最容易在赋值时发生：

```
function foo() {
	console.log( this.a );
} 
var a = 2;
var o = { a: 3, foo: foo };
var p = { a: 4 }; 
o.foo(); // 3 
(p.foo = o.foo)(); // 2
```

赋值表达式 p.foo = o.foo 的返回值是目标函数的引用，因此调用位置是 foo() 而不是 p.foo() 或者 o.foo()。根据我们之前说过的，这里会应用默认绑定。 

注意：<u>对于默认绑定来说，决定 this 绑定对象的并不是调用位置是否处于严格模式，而是 函数体是否处于严格模式。如果函数体处于严格模式，this 会被绑定到 undefined，否则 this 会被绑定到全局对象。</u> 

##### 3）软绑定

之前我们已经看到过，硬绑定这种方式可以把 this 强制绑定到指定的对象（除了使用 new 时），防止函数调用应用默认绑定规则。问题在于，硬绑定会大大降低函数的灵活性，使用硬绑定之后就无法使用隐式绑定或者显式绑定来修改 this。

如果可以给默认绑定指定一个全局对象和 undefined 以外的值，那就可以实现和硬绑定相 同的效果，同时保留隐式绑定或者显式绑定修改 this 的能力。 

可以通过一种被称为软绑定的方法来实现我们想要的效果：

```
if (!Function.prototype.softBind) { 
	Function.prototype.softBind = function(obj) { 
		var fn = this; 
		// 捕获所有 curried 参数
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

除了软绑定之外，softBind(..) 的其他原理和 ES5 内置的 bind(..) 类似。它会对指定的函 数进行封装，首先检查调用时的 this，如果 this 绑定到全局对象或者 undefined，那就把 指定的默认对象 obj 绑定到 this，否则不会修改 this。

下面我们看看 softBind 是否实现了软绑定功能：

```
function foo() { 
	console.log("name: " + this.name); 
}
var obj = { name: "obj" }, 
	obj2 = { name: "obj2" }, 
	obj3 = { name: "obj3" };
	
var fooOBJ = foo.softBind( obj ); 

fooOBJ(); // name: obj 

obj2.foo = foo.softBind(obj); 
obj2.foo(); // name: obj2 <---- 看！！！

fooOBJ.call( obj3 ); // name: obj3 <---- 看！ 

setTimeout( obj2.foo, 10 ); // name: obj <---- 应用了软绑定
```

可以看到，软绑定版本的 foo() 可以手动将 this 绑定到 obj2 或者 obj3 上，但如果应用默 认绑定，则会将 this 绑定到 obj。

### 6.箭头函数

箭头函数并不是使用 function 关键字定义的，而是使用被称为“胖箭头”的操作符 => 定 义的。箭头函数不使用 this 的四种标准规则，而是根据外层（函数或者全局）作用域来决 定 this。 

我们来看看箭头函数的词法作用域：

```
function foo() { 
    // 返回一个箭头函数
    return (a) => { 
    	//this 继承自 foo() 
    	console.log( this.a ); 
    }; 
}

var obj1 = { 
	a:2 
};

var obj2 = { 
	a:3 
};

var bar = foo.call( obj1 ); 
bar.call( obj2 ); // 2, 不是 3 ！
```

foo() 内部创建的箭头函数会捕获调用时 foo() 的 this。由于 foo() 的 this 绑定到 obj1， bar（引用箭头函数）的 this 也会绑定到 obj1，<u>箭头函数的绑定无法被修改。（new 也不 行！）</u>

箭头函数最常用于回调函数中，例如事件处理器或者定时器：

```
function foo() { 
	setTimeout(() => { 
        // 这里的 this 在此法上继承自 foo() 
        console.log( this.a ); },
	100); 
}
var obj = { 
	a:2 
};
foo.call( obj ); // 2
```

箭头函数可以像 bind(..) 一样确保函数的 this 被绑定到指定对象，此外，其重要性还体现在它用更常见的词法作用域取代了传统的 this 机制。实际上，在 ES6 之前我们就已经 在使用一种几乎和箭头函数完全一样的模式。

```
function foo() {
	var self = this; // lexical capture of this 
	setTimeout( function(){ 
		console.log( self.a ); 
	}, 100 ); 
}
var obj = { 
	a: 2 
};
foo.call( obj ); // 2 
```

虽然 self = this 和箭头函数看起来都可以取代 bind(..)，但是从本质上来说，它们想替代的是 this 机制。 

如果你经常编写 this 风格的代码，但是绝大部分时候都会使用 self = this 或者箭头函数 来否定 this 机制，那你或许应当： 

1. 只使用词法作用域并完全抛弃错误 this 风格的代码； 

2. 完全采用 this 风格，在必要时使用 bind(..)，尽量避免使用 self = this 和箭头函数。 

当然，包含这两种代码风格的程序可以正常运行，但是在同一个函数或者同一个程序中混 合使用这两种风格通常会使代码更难维护，并且可能也会更难编写。

**箭头函数注意点：**

（1）函数体内的`this`对象，就是定义时所在的对象，而不是使用时所在的对象。

（2）不可以当作构造函数，也就是说，不可以使用`new`命令，否则会抛出一个错误。

（3）不可以使用`arguments`对象，该对象在函数体内不存在。如果要用，可以用 rest 参数代替。

（4）不可以使用`yield`命令，因此箭头函数不能用作 Generator 函数。

上面四点中，第一点尤其值得注意。`this`对象的指向是可变的，但是在箭头函数中，它是固定的。

