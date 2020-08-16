# vue常见面试题

### 1.响应式数据的原理

在组件`new Vue()`后的执行`vm._init()`初始化过程中，当执行到`initState(vm)`时就会对内部使用到的一些状态，如`props`、`data`、`computed`、`watch`、`methods`分别进行初始化，再对`data`进行初始化的最后有这么一句：

```js
function initData(vm) {  //初始化data
  ...
  observe(data) //  info:{name:'cc',sex:'man'}
}
```

这个`observe`就是将用户定义的`data`变成响应式的数据，接下来看下它的创建过程：

![image-20200718133327835](https://raw.githubusercontent.com/ahaMOMO/autumn-stroke/master/img/20200718133330.png)

![









image-20200718152507694](https://raw.githubusercontent.com/ahaMOMO/autumn-stroke/master/img/20200718152508.png)

- 利用`Proxy`或`Object.defineProperty`生成的Observer针对对象/对象的属性进行"劫持",在属性发生变化后通知订阅者
- 解析器Compile解析模板中的`Directive`(指令)，初始化watcher。收集指令所依赖的方法和数据,等待数据变化然后进行渲染
- Watcher属于Observer和Compile桥梁,它将接收到的Observer产生的数据变化并执行相应的更新函数。（它分为三类：用户watcher(this.$watcher())、计算watcher(computed)和渲染watcher(data中的数据)）;

之后具体内容可参考：https://juejin.im/post/5d4ad8686fb9a06b2766b625#heading-0

总结如下：（响应式原理/数据更新流程）

vue中的数据一开始通过Object.defineProperty对viewModel中数据对象进行属性的get()和set()进行了监听。所以当我们的数据发生更新时，就会触发响应式数据的set方法。当赋值触发`set`时，首先会检测新值和旧值，若相同则直接返回，否则将新值赋值给旧值；如果新值是对象则将它变成响应式之后（调用observe方法）再赋值过去；最后让对应属性的依赖管理器使用`dep.notify`发出更新视图的通知（将收集起来的`watcher`挨个遍历触发`update`方法）。

执行update方法时，将当前watcher实例传入一个watcher队列中，这个队列的作用是将要执行更新的`watcher`收集到一个队列`queue`之内，保证如果同一个`watcher`内触发了多次更新，只会更新一次对应的`watcher`。比如在同一个watcher中对多个属性进行赋值，因为这几个属性它们收集的都是同一个渲染watcher，所以将当前watcher推入队列之后，之后触发的watcher不会再添加进去了。

> 通过这里大家也看出来了，派发更新通知的粒度是组件级别，至于组件内是哪个属性赋值了，派发更新并不关心，而且怎么高效更新这个视图，那是之后`diff`比对做的事情。

队列有了以后，就会执行nextTick方法，这里的nextTick就是我们经常使用的`this.$nextTick`方法。在nextTick方法内部会将队列进行一次排序（computed watcher->user watcher->render watcher,这个顺序可以从它们的初始化顺序就能看出来）。

之后就是遍历这个队列。因为是渲染`watcher`，所以会触发`beforeUpdate`钩子。最后执行`watcher.run()`方法，执行真正的派发更新方法。执行run的时候，会先将模板传入render方法解析成vnode，之后将新旧vnode传入update方法中。vm._update中会先判断是否为首次渲染，如果为首次渲染则调用patch()直接将新vnode渲染为真实dom插入到根节点之内即可。若为重新渲染，则调用patch()对比新旧vnode（diff算法）,最后给真实的dom打补丁。最后触发update钩子，页面得到更新。

### 2.基于`Object.defineProperty`响应式系统的一些不足

只能监听到数据的变化，对于数组或者对象的增加以及删除操作并不能进行监听。

```js
export default {
  data() {
    return {
      arr: [1, 2],
      obj1: {
            a: 3
        }
    }
  },
  methods: {
    editInfo() {  
      	this.obj1.b = 'man'; //不会重新渲染
      	this.arr[0] = 3; 	//不会重新渲染
        
        //以下方法可以实现
      	Vue.set(this.arr,0,3);
        this.$set(this.arr,0,3);
    },
  }
}
```

数据是被赋值了，但是视图并不会发生变更。`vue`为了解决这个问题，提供了两个`API`：`$set`和`$delete`，它们又是怎么办到的了？

##### Vue.set()和this.$set()实现原理

Vue.set()是将set函数绑定在Vue构造函数上，this.$set()是将set函数绑定在Vue原型上。

- 如果是在开发环境，且target未定义（为null、undefined）或target为基础数据类型（string、boolean、number、symbol）时，抛出告警；
- 如果target为数组且key为有效的数组key时，将数组的长度设置为target.length和key中的最大的那一个，然后**调用数组的splice方法**（vue中重写的splice方法）添加元素；
- 如果属性key存在于target对象中且key不是Object.prototype上的属性时，表明这是在修改target对象属性key的值（不管target对象是否是响应式的，只要key存在于target对象中，就执行这一步逻辑），此时就直接将value直接赋值给target[key]；
- 判断target，当target为vue实例或根数据data对象时，在开发环境下抛错；
- 当一个数据为响应式时，vue会给该数据添加一个`__ob__`属性，因此可以通过判断target对象是否存在`__ob__`属性来判断target是否是响应式数据，当target是非响应式数据时，我们就按照普通对象添加属性的方式来处理；当target对象是响应式数据时，我们将target的属性key也设置为响应式并手动触发通知其属性值的更新；

### 3.`Object.defineProperty`和`proxy`的区别在哪

##### Object.defineProperty实现双向数据绑定的缺陷：

- 无法监听数组的变化
- 只能劫持对象的属性,因此我们需要对每个对象的每个属性进行遍历，如果属性值也是对象那么需要深度遍历，所以我们动态添加对象的属性也是无法进行响应式监听的

##### Proxy实现双向数据绑定的优点：

- Proxy可以直接监听对象而非属性
- Proxy可以直接监听数组的变化
- Proxy有多达13种拦截方法,不限于apply、ownKeys、deleteProperty、has等等是`Object.defineProperty`不具备的
- Proxy返回的是一个新对象,我们可以只操作新的对象达到目的,而`Object.defineProperty`只能遍历对象属性直接修改

##### Proxy实现双向数据绑定的缺点：

- 兼容性没有那么好

### 4.如何监听一个数组的变化？

> Vue.js 包装了被观察数组的变异方法，故它们能触发视图更新。被包装的方法有：
>
> - push()
> - pop()
> - shift()
> - unshift()
> - splice()
> - sort()
> - reverse()

> Vue.js 不能检测到下面数组变化：
>
> - 直接用索引设置元素，如 vm.items[0] = {}；
>
>   解决方法：
>
>   - vm.items.splice(indexOfItem,1,newValue);
>   - vm.$set(vm,items,indexOfItem,newValue);
>
> - 修改数据的长度，如 vm.items.length = 0。
>
>   解决方法：
>
>   - vm.items.splice(newLength)；

为什么只有这些特定的方法才能触发视图的更新呢？

整体思路是：vue通过重新包装数据中数组的push、pop等常用方法。这里的包装指的是数据数组（也就是我们要监听的数组，xue实例中拥有的data数据）的方法。而不是改变js原生Array中的原型方法。

代码实现：

```js
const aryMethods = ['push', 'pop', 'shift', 'unshift', 'splice', 'sort', 'reverse'];
const arrayAugmentations = [];

aryMethods.forEach((method)=> {

    // 这里是原生Array的原型方法
    let original = Array.prototype[method];

   // 将push, pop等封装好的方法定义在对象arrayAugmentations的属性上
   // 注意：是属性而非原型属性
    arrayAugmentations[method] = function () {
        console.log('我被改变啦!');

        // 调用对应的原生方法并返回结果
        return original.apply(this, arguments);
    };

});

let list = ['a', 'b', 'c'];
// 将我们要监听的数组的原型指针指向上面定义的空数组对象
// 别忘了这个空数组的属性上定义了我们封装好的push等方法
list.__proto__ = arrayAugmentations;
list.push('d');  // 我被改变啦！ 4

// 这里的list2没有被重新定义原型指针，所以就正常输出
let list2 = ['a', 'b', 'c'];
list2.push('d');  // 4
```

### 5.vue-router路由模式有几种？原理是什么？

##### hash路由

> hash路由一个明显的标志是带有`#`,我们主要是通过监听url中的hash变化来进行路由跳转。
>
> hash的优势就是兼容性更好,在老版IE中都有运行,问题在于url中一直存在`#`不够美观,而且hash路由更像是Hack而非标准,相信随着发展更加标准化的**History API**会逐步蚕食掉hash路由的市场。

代码实现：

```HTML
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>hash router</title>
</head>
<body>
  <ul>
      <li><a href="#/">turn yellow</a></li>
      <li><a href="#/blue">turn blue</a></li>
      <li><a href="#/green">turn green</a></li>
  </ul>
  <button>back</button>
  <script src="./hash.js" charset="utf-8"></script>
</body>
</html>
```

```js
class Routers {
  constructor() {
    // 储存hash与callback键值对
    this.routes = {};
    // 当前hash
    this.currentUrl = '';
    // 记录出现过的hash
    this.history = [];
    // 作为指针,默认指向this.history的末尾,根据后退前进指向history中不同的hash
    this.currentIndex = this.history.length - 1;
    this.refresh = this.refresh.bind(this);
    this.backOff = this.backOff.bind(this);
    // 默认不是后退操作
    this.isBack = false;
    window.addEventListener('load', this.refresh, false);
    window.addEventListener('hashchange', this.refresh, false);
  }

  route(path, callback) {
    this.routes[path] = callback || function() {};
  }

  refresh() {
    this.currentUrl = location.hash.slice(1) || '/';
    if (!this.isBack) {
      // 如果不是后退操作,且当前指针小于数组总长度,直接截取指针之前的部分储存下来
      // 此操作来避免当点击后退按钮之后,再进行正常跳转,指针会停留在原地,而数组添加新hash路由
      // 避免再次造成指针的不匹配,我们直接截取指针之前的数组
      // 此操作同时与浏览器自带后退功能的行为保持一致
      if (this.currentIndex < this.history.length - 1)
        this.history = this.history.slice(0, this.currentIndex + 1);
      this.history.push(this.currentUrl);
      this.currentIndex++;
    }
    this.routes[this.currentUrl]();
    console.log('指针:', this.currentIndex, 'history:', this.history);
    this.isBack = false;
  }
  // 后退功能
  backOff() {
    // 后退操作设置为true
    this.isBack = true;
    this.currentIndex <= 0
      ? (this.currentIndex = 0)
      : (this.currentIndex = this.currentIndex - 1);
    location.hash = `#${this.history[this.currentIndex]}`;
    this.routes[this.history[this.currentIndex]]();
  }
}

window.Router = new Routers();
const content = document.querySelector('body');
const button = document.querySelector('button');
function changeBgColor(color) {
  content.style.backgroundColor = color;
}

Router.route('/', function() {
  changeBgColor('yellow');
});
Router.route('/blue', function() {
  changeBgColor('blue');
});
Router.route('/green', function() {
  changeBgColor('green');
});

button.addEventListener('click', Router.backOff, false);
```

##### History API(HTML5 新路由方案)

> 常用的API
>
> ```JS
> window.history.back();       // 后退 
> window.history.forward();    // 前进
> window.history.go(-3);       // 后退三个页面
> ```

`history.pushState`用于在浏览历史中添加历史记录,但是并不触发跳转,此方法接受三个参数，依次为：

- `state`:一个与指定网址相关的状态对象，`popstate`事件触发时，该对象会传入回调函数。如果不需要这个对象，此处可以填`null`。
-  `title`：新页面的标题，但是所有浏览器目前都忽略这个值，因此这里可以填`null`。
-  `url`：新的网址，必须与当前页面处在同一个域。浏览器的地址栏将显示这个网址。

`history.replaceState`方法的参数与`pushState`方法一模一样，区别是它修改浏览历史中当前纪录,而非添加记录,同样不触发跳转。

`popstate`事件,每当同一个文档的浏览历史（即history对象）出现变化时，就会触发popstate事件。

需要注意的是，仅仅调用`pushState`方法或`replaceState`方法 ，并不会触发该事件，只有用户点击浏览器倒退按钮和前进按钮，或者使用 JavaScript 调用`back`、`forward`、`go`方法时才会触发。另外，该事件只针对同一个文档，如果浏览历史的切换，导致加载不同的文档，该事件也不会触发。

```HTML
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>h5 router</title>
</head>
<body>
  <ul>
      <li><a href="/">turn yellow</a></li>
      <li><a href="/blue">turn blue</a></li>
      <li><a href="/green">turn green</a></li>
  </ul>
</body>
</html>

```

```JS
class Routers {
  constructor() {
    this.routes = {};
    this._bindPopState();
  }
  init(path) {
    history.replaceState({path: path}, null, path);
    this.routes[path] && this.routes[path]();
  }

  route(path, callback) {
    this.routes[path] = callback || function() {};
  }

  go(path) {
    history.pushState({path: path}, null, path);
    this.routes[path] && this.routes[path]();
  }
  _bindPopState() {
    window.addEventListener('popstate', e => {
      const path = e.state && e.state.path;
      this.routes[path] && this.routes[path]();
    });
  }
}

window.Router = new Routers();
Router.init(location.pathname);
const content = document.querySelector('body');
const ul = document.querySelector('ul');
function changeBgColor(color) {
  content.style.backgroundColor = color;
}

Router.route('/', function() {
  changeBgColor('yellow');
});
Router.route('/blue', function() {
  changeBgColor('blue');
});
Router.route('/green', function() {
  changeBgColor('green');
});

ul.addEventListener('click', e => {
  if (e.target.tagName === 'A') {
    e.preventDefault();
    Router.go(e.target.getAttribute('href'));
  }
});

```

### 6.computed和watch的区别和原理

##### computed原理：

我们需要理解以下问题：

- computed是如何初始化，初始化之后干了些什么
- 为何触发data值改变时computed会重新计算
- computed值为什么说是被缓存的呢，如何做的

##### watch原理：

其实主要就是分析计算属性为何可以做到当它的依赖项发生改变时才会进行重新的计算，否则当前数据是被缓存的。

##### 两者区别：

> computed：是计算属性，依赖其它属性值，并且 computed 的值有缓存，只有它依赖的属性值发生改变，下一次获取 computed 的值时才会重新计算 computed 的值；<u>当模板中的某个值需要通过一个或多个数据计算得到时，就可以使用计算属性</u>，还有计算属性的函数不接受参数；
>
> watch：主要是监听某个值发生变化后，对新值去进行逻辑处理。类似于某些数据的监听回调 ，每当监听的数据变化时都会执行回调进行后续操作；当我们需要在数据变化时执行异步或开销较大的操作时，应该使用 watch，使用 watch 选项允许我们执行异步操作 (访问一个API)，限制我们执行该操作的频率，并在我们得到最终结果前，设置中间状态。这些都是计算属性无法做到的。

### 7.当前组件模板中用到的变量一定要定义在`data`里么？

`data`中的变量都会被代理到当前`this`下，所以我们也可以在`this`下挂载属性，只要不重名即可。而且定义在`data`中的变量在`vue`的内部会将它包装成响应式的数据，让它拥有变更即可驱动视图变化的能力。但是如果这个数据不需要驱动视图，定义在`created`或`mounted`钩子内也是可以的，因为不会执行响应式的包装方法，对性能也是一种提升。

### 8.vue中的key有什么用？

key 是为 Vue 中 vnode 的唯一标记，通过这个 key，我们的 diff 操作可以更准确、更快速。Vue 的 diff 过程可以概括为：oldCh 和 newCh 各有两个头尾的变量 oldStartIndex、oldEndIndex 和 newStartIndex、newEndIndex，它们会新节点和旧节点会进行两两对比，即一共有4种比较方式：newStartIndex 和oldStartIndex 、newEndIndex 和 oldEndIndex 、newStartIndex 和 oldEndIndex 、newEndIndex 和 oldStartIndex，如果以上 4 种比较都没匹配，如果设置了key，就会用 key 再进行比较，在比较的过程中，遍历会往中间靠，一旦 StartIdx > EndIdx 表明 oldCh 和 newCh 至少有一个已经遍历完了，就会结束比较。

### 9.vue3.0新特性

- vue3重新审视了vdom,更改了自身对于vdom的对比算法。vdom从之前的每次更新，都进行一次完整的对比，改为了切分区块树，来进行动态内容更新。也就是只更新vdom的绑定了动态数据的部分，把速度提高了6倍。
- 把definePerproty改为了proxy,对于js引擎更加友好，响应更加高效。
- 之前vue的代码，只有一-个vue对象进来，所有的东西都在vue上,这样的话其实所有你没用到的东西也没有办法扔掉，因为它们全都已经被添加到vue这个全局对象上了。vue3的话，一些不是每个应用都需要的功能，我们就做成了按需引入。用ES module imports按需引入，举例来说，内置组件像keep-alive、transition, 指令的工具函数。比如async component、使用mixins、或者是memoize都可以做成按需引入。
- 加强了typescript的支持，虽然我们在vue2已经可以使用typescript了,但是在vue3中，进一步加强了对typescript的支持， 很可能以后你就需要用typescript来写vue了;

### 10.vue指令有哪些？v-if/v-show的区别

![img](https://raw.githubusercontent.com/ahaMOMO/autumn-stroke/master/img/20200804153421)

### 11.vue是什么？怎么理解？

vuejs是一种前端的渐进式js框架。它让我们只需要关注视图层，十分容易上手。

它有两大核心思想：

1）数据驱动：视图是由数据驱动生成的，我们对视图的修改，不会直接操作 DOM，而是通过修改数据。它相比我们传统的前端开发，如使用 jQuery 等前端库直接修改 DOM，大大简化了代码量。特别是当交互复杂的时候，只关心数据的修改会让代码的逻辑变的非常清晰，因为 DOM 变成了数据的映射，我们所有的逻辑都是对数据的修改，而不用碰触 DOM，这样的代码非常利于维护。

> 数据驱动中讲的是怎么把原始数据映射到DOM中，以及数据变化到DOM变化的部分，而数据变化到DOM变化的部分依靠的就是它的响应式原理。

2）组件化：把页面拆分成多个组件 (component)，每个组件依赖的 CSS、JavaScript、模板、图片等资源放在一起开发和维护。组件是资源独立的，组件在系统内部可复用，组件和组件之间可以嵌套。

### 12.vue-cli2.0和vue-cli3.0的区别？其中vue-cli3.0的vue.config.js用来做什么？

参考：https://gitpress.io/@rainy/vue-cli3

### 13.emit/on实现

```js
class Event {
    constructor() {
        this._events = {}; //装载事件
    }

    on(eventName, callback) {
        if (!this._events) {
            this._events = {};
        }
        if (!this._events[eventName]) {
            this._events[eventName] = [];
        }
        this._events[eventName].push(callback);

    }

    emit(eventName, ...args) {
        let handle = this._events[eventName];
        if (handle) {
            for (let i = 0; i < handle.length; i++) {
                handle[i].apply(this, args);
            }
        }
    }
    off(eventName,fn){
        let handle = this._events[eventName];
        if (handle) {
            let position = -1; //记录该函数所在位置
            for (let i = 0; i < handle.length; i++) {
                if(handle[i] === fn){
                    position = i;
                }
            }
            if(position != -1){
                handle.splice(position,1);
            }
        }
    }
}

let event = new Event();
event.on("b", function (...msg) {
    console.log(msg);
});
event.emit("b", "您好","Sdf");
```





