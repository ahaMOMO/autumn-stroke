组件可以有以下几种关系：

![image-20200802091711307](https://raw.githubusercontent.com/ahaMOMO/autumn-stroke/master/img/20200802091721.png)

如上图所示，A 和 B、B 和 C、B 和 D 都是父子关系，C 和 D 是兄弟关系，A 和 C 是隔代关系（可能隔多代）。

针对不同的使用场景，如何选择行之有效的通信方式？

### 一、props和emit（父子组件通信）

父组件通过props向下传递数据给子组件，子组件通过events给父组件发送消息，实际上就是子组件把自己的数据发送到父组件。

例子：

```vue
//App.vue父组件
<template>
  <div id="app">
    <users v-bind:users="users" v-on:titleChanged="updateTitle"></users>//前者自定义名称便于子组件调用，后者要传递数据名
  </div>
</template>

<script>
import Users from "./components/Users"
export default {
  name: 'App',
  data(){
    return{
      users:["Henry","Bucky","Emily"]
    }
  },
  components:{
    "users":Users  //注册子组件
  },
  methods:{
    updateTitle(arg){   //声明这个函数
      //执行相关操作
    }
  }
}
</script>
```

```vue
//users子组件
<template>
  <div class="hello">
    <h1 @click="changeTitle">{{title}}</h1>//绑定一个点击事件
    <ul>
      <li v-for="user in users">{{user}}</li>//遍历传递过来的值，然后呈现到页面
    </ul>
  </div>
</template>
<script>
export default {
  name: 'HelloWorld',
  props:{
    users:{           //这个就是父组件中子标签自定义名字(对应v-bind:users)
      type:Array,
      required:true
    }
  },
  data() {
    return {
      title:"Vue.js Demo"
    }
  },
  methods:{
    changeTitle() {
      this.$emit("titleChanged","子向父组件传值");//自定义事件  传递值“子向父组件传值”
    }
  }
}
</script>
```

##### 扩展内容：

###### 1.子组件可以改变父组件的数据吗？为什么？

为了保证数据的单向流动，便于对数据进行追踪，避免数据混乱。

所有的 prop 都使得其父子 prop 之间形成了一个**单向下行绑定**：父级 prop 的更新会向下流动到子组件中，但是反过来则不行。这样会防止从子组件意外变更父级组件的状态，从而导致你的应用的数据流向难以理解。

额外的，每次父级组件发生变更时，子组件中所有的 prop 都将会刷新为最新的值。这意味着你不应该在一个子组件内部改变 prop。如果你这样做了，Vue 会在浏览器的控制台中发出警告。

注意在 JavaScript 中对象和数组是通过引用传入的，所以对于一个数组或对象类型的 prop 来说，在子组件中改变变更这个对象或数组本身将会影响到父组件的状态。

```js
 function initProps (vm: Component, propsOptions: Object) {
     //获取父组件传入的props对象。
     const propsData = vm.$options.propsData || {}
     /*这里没有用 defineReactive 函数直接处理 propsDatas, 而是用一个新变量来接受props来接受 
          defineReactive的处理 */
     const props = vm._props = {}
     const keys = vm.$options._propKeys = []
     const isRoot = !vm.$parent

     if (!isRoot) {
         toggleObserving(false)
     }
     for (const key in propsOptions) {
         keys.push(key)
         const value = validateProp(key, propsOptions, propsData, vm)

         if (process.env.NODE_ENV !== 'production') {
             const hyphenatedKey = hyphenate(key)
             if (isReservedAttribute(hyphenatedKey) ||
                 config.isReservedAttr(hyphenatedKey)) {
                 warn(
                     `"${hyphenatedKey}" is a reserved attribute and cannot be used as component prop.`,
                     vm
                 )
             }
             defineReactive(props, key, value, () => {
                 if (!isRoot && !isUpdatingChildComponent) {
                     //当存在父组件并且修改来源于子组件的时候给出警告
                     warn(
                         `Avoid mutating a prop directly since the value will be ` +
                         `overwritten whenever the parent component re-renders. ` +
                         `Instead, use a data or computed property based on the prop's ` +
                         `value. Prop being mutated: "${key}"`,
                         vm
                     )
                 }
             })
         } else {
             defineReactive(props, key, value)
         }

         if (!(key in vm)) {
             /*把_props的属性经过属性代理 ，方便我们可以用 this[key] 直接访问 vm下边的_props 里边的属性*/
             proxy(vm, `_props`, key)
         }
     }
 }
```

分析：

在组件进行initProps方法的时候，会执行defineReactive方法，这个方法就是运行Object.defineProperty对传入的object绑定get/set，传入的第四个参数是触发set的回调。所以props被修改时，就会查看是不是父组件、是不是更新子组件，如果不是父组件并且不是更新子组件，那说明是子组件在修改props，给出warn警告。

> 然而 props传入的是对象的话 是可以直接在子组件里更改的, 因为是同一个引用
> 组件对于data的监听是深度监听
> 而对于props的监听是浅度监听

###### 2.父组件的数据更改是怎么通知到所有的子组件的？

首先，`prop` 数据的值变化在父组件，我们知道在父组件的 `render` 过程中会访问到这个 `prop` 数据，所以当 `prop` 数据变化一定会触发父组件的重新渲染，那么重新渲染是如何更新子组件对应的 `prop` 的值呢？

在父组件重新渲染的最后，会执行 `patch` 过程，进而执行 `patchVnode` 函数，`patchVnode` 通常是一个递归过程，当它遇到组件 `vnode` 的时候，会执行组件更新过程的 `prepatch` 钩子函数。其内部会调用 `updateChildComponent` 方法来更新 `props`。

- 如果是基本类型，是这个流程：

  父组件数据改变，只会把新的数据传给子组件。

  子组件拿到新数据，就会直接替换到原来的 props。

  而 props 在子组件中也是响应式的，【直接 等号 替换】导致触发 set，set 再通知 子组件完成更新。

  > watcher1 是父组件，watcher2 是子组件
  >
  > 父组件内的 data num 通知 watcher1 更新
  > 子组件内的 props child_num 通知 watcher2 更新

- 如果是对象，是这个流程:

  父组件传 对象 给 子组件，并且父子组件 页面都使用到了这个数据.

  那么这个对象，会收集到 父子组件的 watcher

  当 对象内部被修改的时候，会通知到 父和子 更新。

> **区别是什么？**
>
> 1、基本类型是，子组件内部 props 通知 子组件更新的
>
> 2、引用类型是，父组件的数据 data 通知 子组件更新的

### 二、$parent/$children与ref(适合父子组件)

- `ref`：如果在普通的 DOM 元素上使用，引用指向的就是 DOM 元素；如果用在子组件上，引用就指向组件实例
- `$parent` / `$children`：访问父 / 子实例

需要注意的是：这两种都是直接得到组件实例，使用后可以直接调用组件的方法或访问数据。我们先来看个用 `ref`来访问组件的例子：

```js
// component-a 子组件
export default {
  data () {
    return {
      title: 'Vue.js'
    }
  },
  methods: {
    sayHello () {
      window.alert('Hello');
    }
  }
}
```

```js
// 父组件
<template>
  <component-a ref="comA"></component-a>
</template>
<script>
  export default {
    mounted () {
      const comA = this.$refs.comA;
      console.log(comA.title);  // Vue.js
      comA.sayHello();  // 弹窗
    }
  }
</script>
```

### 三、$attrs和$listeners（适合隔代通信）

若只是a->c组件单纯进行简单的数据传递，而且不要求子组件往上（父方向）传递数据。则可以使用这个。

- `$attrs`：包含了父作用域中不被 props 所识别 (且获取) 的特性绑定 (class 和 style 除外)。当一个组件没有声明任何 prop 时，这里会包含所有父作用域的绑定 (class 和 style 除外)，并且可以通过 v-bind="$attrs" 传入内部组件。通常配合 inheritAttrs 选项一起使用。
- `$listeners`：包含了父作用域中的 (不含 .native 修饰器的) v-on 事件监听器。它可以通过 v-on="$listeners" 传入内部组件。

例子：

```vue
// index.vue
<template>
  <div>
    <h2>浪里行舟</h2>
    <child-com1
      :foo="foo"
      :boo="boo"
      :coo="coo"
      :doo="doo"
      title="前端工匠"
      @click.native="say" 
      @mouseover="sing"
    ></child-com1>
  </div>
</template>
<script>
const childCom1 = () => import("./childCom1.vue");
export default {
  components: { childCom1 },
  data() {
    return {
      foo: "Javascript",
      boo: "Html",
      coo: "CSS",
      doo: "Vue"
    };
  },
  methods: {
    say() {},
    sing() {}
  }
};
</script>
```

```vue
// childCom1.vue
<template class="border">
  <div>
    <p>foo: {{ foo }}</p>
    <p>childCom1的$attrs: {{ $attrs }}</p>
    <child-com2 v-bind="$attrs" v-on="$listeners"></child-com2>
  </div>
</template>
<script>
const childCom2 = () => import("./childCom2.vue");
export default {
  components: {
    childCom2
  },
  inheritAttrs: false, // 可以关闭自动挂载到组件根元素上的没有在props声明的属性
  props: {
    foo: String // foo作为props属性绑定
  },
  created() {
    console.log(this.$attrs); // { "boo": "Html", "coo": "CSS", "doo": "Vue", "title": "前端工匠" }
    console.log(this.$listeners);  //{mouseover: ƒ}
  }
};
</script>
```

```vue
// childCom2.vue
<template>
  <div class="border">
    <p>boo: {{ boo }}</p>
    <p>childCom2: {{ $attrs }}</p>
  </div>
</template>
<script>
export default {
  inheritAttrs: false,
  props: {
    boo: String
  },
  created() {
    console.log(this.$attrs); // { "coo": "CSS", "doo": "Vue", "title": "前端工匠" }
    console.log(this.$listeners);  //{mouseover: ƒ}
  }
};
</script>
```

### 四、provide/inject(适合父子、隔代通信)

Vue2.2.0新增API,这对选项需要一起使用，**以允许一个祖先组件向其所有子孙后代注入一个依赖，不论组件层次有多深，并在起上下游关系成立的时间里始终生效**。一言而蔽之：祖先组件中通过provider来提供变量，然后在子孙组件中通过inject来注入变量。 **provide / inject API 主要解决了跨级组件间的通信问题，不过它的使用场景，主要是子组件获取上级组件的状态，跨级组件间建立了一种主动提供与依赖注入的关系**。

例子：

假设有两个组件： A.vue 和 B.vue，B 是 A 的子组件

```js
// A.vue
export default {
  provide: {
    name: '浪里行舟'
  }
}
```

```js
// B.vue
export default {
  inject: ['name'],
  mounted () {
    console.log(this.name);  // 浪里行舟
  }
}
```

可以看到，在 A.vue 里，我们设置了一个 **provide: name**，值为 浪里行舟，它的作用就是将 **name** 这个变量提供给它的所有子组件。而在 B.vue 中，通过 `inject` 注入了从 A 组件中提供的 **name** 变量，那么在组件 B 中，就可以直接通过 **this.name** 访问这个变量了，它的值也是 浪里行舟。这就是 provide / inject API 最核心的用法。

需要注意的是：**provide 和 inject 绑定并不是可响应的。这是刻意为之的。然而，如果你传入了一个可监听的对象，那么其对象的属性还是可响应的**。所以，上面 A.vue 的 name 如果改变了，B.vue 的 this.name 是不会改变的。

> 不管子组件多深，只要调用了inject就可以注入

##### 扩展内容：

###### 1.project与inject怎么实现数据响应式？

一般来说，有两种办法：

- provide里面提供祖先组件的实例，然后在子孙组件中注入依赖，这样就可以在子孙组件中直接修改祖先组件的实例的属性，不过这种方法有个缺点就是这个实例上挂载很多没有必要的东西比如props，methods
- 使用2.6最新API Vue.observable 优化响应式 provide(推荐)

我们来看个例子：孙组件D、E和F获取A组件传递过来的color值，并能实现数据响应式变化，即A组件的color变化后，组件D、E、F会跟着变（核心代码如下：）

![image-20200802123131177](https://raw.githubusercontent.com/ahaMOMO/autumn-stroke/master/img/20200802123132.png)

```js
// A 组件 
<div>
      <h1>A 组件</h1>
      <button @click="() => changeColor()">改变color</button>
      <ChildrenB />
      <ChildrenC />
</div>
......
  data() {
    return {
      color: "blue"
    };
  },
  // provide() {
  //   return {
  //     theme: {
  //       color: this.color //这种方式绑定的数据并不是可响应的
  //     } // 即A组件的color变化后，组件D、E、F不会跟着变
  //   };
  // },
  provide() {
    return {
      theme: this//方法一：提供祖先组件的实例
    };
  },
  methods: {
    changeColor(color) {
      if (color) {
        this.color = color;
      } else {
        this.color = this.color === "blue" ? "red" : "blue";
      }
    }
  }
  // 方法二:使用2.6最新API Vue.observable 优化响应式 provide
  // provide() {
  //   this.theme = Vue.observable({
  //     color: "blue"
  //   });
  //   return {
  //     theme: this.theme
  //   };
  // },
  // methods: {
  //   changeColor(color) {
  //     if (color) {
  //       this.theme.color = color;
  //     } else {
  //       this.theme.color = this.theme.color === "blue" ? "red" : "blue";
  //     }
  //   }
  // }
```

```vue
// F 组件 
<template functional>
  <div class="border2">
    <h3 :style="{ color: injections.theme.color }">F 组件</h3>
  </div>
</template>
<script>
export default {
  inject: {
    theme: {
      //函数式组件取值不一样
      default: () => ({})
    }
  }
};
</script>
```

### 五、$emit和$on（适合父子、隔代、兄弟通信）

这种方法通过一个空的Vue实例作为中央事件总线（事件中心），用它来触发事件和监听事件,巧妙而轻量地实现了任何组件间的通信，包括父子、兄弟、跨级。

例子：（实现childa,childb组件传参）

```vue
var Event=new Vue();
Event.$emit(事件名,数据);
Event.$on(事件名,data => {});
```

```vue
//父组件
<template>
     <div>
      <childa></childa>
      <br />
      <childb></childb>    
     </div>
</template>
<script>
   import childa from './childa.vue';
   import childb from './childb.vue';
   export default {
    components:{
        childa,
        childb
    },
    data(){
        return {
            msg:""
        }
    },
    methods:{
       
    }
   }
</script>
```

```vue
//childa组件
<template>
    <div>
        <span>A组件->{{msg}}</span>
        <input type="button" value="把a组件数据传给b" @click ="send">
    </div>
</template>
<script>
import vmson from "../../../util/emptyVue"
export default {
    data(){
        return {
            msg:{
                a:'111',
                b:'222'
            }
        }
    },
    methods:{
        send:function(){
            Event.$emit("aevent",this.msg)
        }
    }
}
</script>
```

```vue
//childb组件
<template>
 <div>
    <span>b组件,a传的的数据为->{{msg}}</span>
 </div>
</template>
<script>
      import vmson from "../../../util/emptyVue"
      export default {
         data(){
                return {
                    msg:""
                }
            },
         mounted(){
                Event.$on("aevent",(val)=>{//监听事件aevent，回调函数要使用箭头函数;
                    console.log(val);//打印结果：我是a组件的数据
                    this.msg = val;
                })
          }
    }
</script>
```

##### 扩展问题：

###### 1.如何实现一个emit/on

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
event.on("b", function (msg) {
    console.log(msg);
});
event.emit("b", "您好");
 ```



### 六、vuex(适合父子、隔代和兄弟通信)