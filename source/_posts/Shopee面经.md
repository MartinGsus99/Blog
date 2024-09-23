# Shopee面经

### 1.手写html的空白页面， style标签可以放在body底部吗，为什么？

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>

<body>
    <header></header>
    <nav></nav>
    <aside>
        <section></section>
    </aside>
    <aside></aside>
    <footer></footer>
</body>

</html>
```

- 放在body底部是可以的，但是可能会引起FOUC、重绘、重排；

### 2.html5的新标签有哪些？

#### 布局标签

- header
- nav
- aritcle
- section
- aside
- footer

#### 多媒体标签

- audio
- video

#### 表单标签

- email
- url
- date
- time
- month
- number
- tel
- color

### 3.了解过bind，call，apply吗，有什么不同？

> 都是改变函数执行时的上下文，简言之就是改变函数的this指向；

何时需要改变this指向？

```js
var name = "lucy";
var obj = {
    name: "martin",
    say: function () {
        console.log(this.name);
    }
};
obj.say(); // martin，this 指向 obj 对象
setTimeout(obj.say,0); // lucy，this 指向 window 对象
```

> setTimeout在定时器中作为回调函数执行，this指向全局上下文；此时this指向window；

appy接受两个参数，一个是this指向，另一个接受函数参数；改变this指向后立即执行，且只是临时改变一次this指向；第一个参数为null、undefined时指向window；

call第一个参数也是`this`的指向，后面传入的是一个参数列表；变this指向后立即执行，且只是临时改变一次this指向；第一个参数为null、undefined时指向window；

bind和call相似，第一个参数为this指向，第二个为参数列表（可以多次传入），不会立即执行，返回一个永久改变this指向的函数；

#### 实现

```js
//call
Function.prototype.myCall = function (context = window, ...args) {
    if (typeof context !== 'object') {
        context = new Object(context)
    }
    let fnkey = Symbol()
    context[fnkey] = this

    // 相当于 obj.fn()执行 fn内部this指向context(obj)
    let res = context[fnkey](...args)
    delete context[fnkey]
    return res

}

function f (a, b) {
    console.log(this.name, a, b)
}

let obj = {
    name: 'Martin'
}

f.myCall(obj, 1, 2)
```

```js
//apply
Function.prototype.myApply=function(context=window,args){
    if(typeof context !=='object'){
        context=new Object(context);
    }
    
    let fnkey=Symbol();
    context[fnkey]=this;
    let res=context[fnkey](...args);
     delete context[fnkey];
    return res;
}
```

```js
Function.prototype.myBind=function(context=window,...args){
    // this表示调用bind的函数
    let self=this;
    
    let fBinded=function(...innerArgs){
        return self.apply(
        	this instanceof fBinded ? this:context,
            args.concat(innerArgs)
        )
    }
    
    fBinded.prototype=Object.create(this.prototype);
    return fBinded;
    
}
```

### 4.跨域知道吗？哪些方式来解决？

> 同源策略SOP Same Origin Policy 协议、域名、端口全部相同；

#### 解决方案

##### jsonp

> 本质是利用scipt标签的开放策略，浏览器传递callback参数到后端，后端返回数据时将callback参数作为函数名来包裹。

```js
var script = document.createElement('script');
script.type = 'text/javascript';
// 传参callback给后端，后端返回时执行这个在前端定义的回调函数
script.src = 'http://a.qq.com/index.php?callback=handleCallback';
document.head.appendChild(script);
// 回调执行函数
function handleCallback(res) {
    alert(JSON.stringify(res));
}

```

优点：

- 兼容性好；

缺点：

- 没有错误处理
- 只支持get
- 无法设置资源访问权限；

##### CORS

> 目前，所有主流浏览器（IE10及以上）使用XMLHttpRequest对象都可支持该功能，IE8和IE9需要使用XDomainRequest对象进行兼容。
>
> CORS整个通信过程都是浏览器自动完成，浏览器一旦发现ajax请求跨源，就会自动在头信息中增加Origin字段，用来说明本次请求来自哪个源（协议+域名+端口）。
>
> 服务端配置：
>
> Access-Control-Allow-Origin 头信息；
>
> 前端设置withCredentials；

![图片](https://i-blog.csdnimg.cn/blog_migrate/3e75dbf4323d2af9d31fb8b2e4a17ba8.png)

```js
// IE8/9需用XDomainRequest兼容
var xhr = new XMLHttpRequest(); 
// 前端设置是否带cookie
xhr.withCredentials = true;
xhr.open('post', 'http://a.qq.com/index.php', true);
xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
xhr.send('user=saramliu');
xhr.onreadystatechange = function() {
    if (xhr.readyState == 4 && xhr.status == 200) {
        alert(xhr.responseText);
    }
};

```

优点：

- 支持所有HTTP请求；
- 有onerror处理错误；
- 可以进行资源访问搜全；

缺点：

- 兼容性差；

##### 服务器代理

> 因为代理服务器使用了CORS，使得浏览器允许前端服务器访问代理服务器返回403；

![图片](https://i-blog.csdnimg.cn/blog_migrate/132872df58d23f3fbb2a63088c3038df.png)

##### vue框架跨域

```js
module.exports = {
    entry: {},
    module: {},
    ...
    devServer: {
        historyApiFallback: true,
        proxy: [{
            context: '/login',
            target: 'http://www.demo2.com:8080',  // 代理跨域目标接口
            changeOrigin: true,
            secure: false,  // 当代理某些https服务报错时用
            cookieDomainRewrite: 'www.demo1.com'  // 可以为false，表示不修改
        }],
        noInfo: true
    }
}

```

##### webSocket协议

> WebSocket protocol是HTML5一种新的协议。它实现了浏览器与服务器全双工通信，同时允许跨域通讯，是server push技术的一种很好的实现。 原生WebSocket API使用起来不太方便，我们使用Socket.io，它很好地封装了webSocket接口，提供了更简单、灵活的接口，也对不支持webSocket的浏览器提供了向下兼容。
>

##### Nginx

```
location / {
  add_header Access-Control-Allow-Origin *;
}

```

### 5.fetch超时后的函数操作，用Promise来写

> fetch网络请求没有timeout机制，也没有abort机制；

```js
let controller = new AbortController()
let signal = controller.signal

const timeoutAction = timer => {
    return new Promise(reslove => {
        setTimeout(() => {
            const response = new Response(
                JSON.stringify({
                    code: 1,
                    msg: `timeout ${timer}s`
                })
            )
            reslove(response)
            controller.abort() // 发送终止信号
        }, timer * 1000)
    })
}
const url = "/data.json" // 请求的url

// 执行，超时控制在0秒，让它永远超时，方便观察
Promise.race([
    timeoutAction(0),
    fetch(url, {
        signal: signal //设置信号
    })
])
    .then(res => {
        return res.json()
    })
    .then(res => {
        if (res.code !== 0) {
            return alert(res.msg)
        } else {
            return console.log(res)
        }
    })
```

### 6.输出

```js
const func1 = x => x
const func2 = x => { x }
const func3 = x => ({ x })

console.log(
    func1(1),
    func2(2),
    func3(3))

//1 undefined { x: 3 }
```

- 箭头函数没有大括号的情况下默认返回表达式的值；
- 有大括号表示代码块，没有返回return；
- 大括号被包裹在圆括号中，表示返回字面量{x:x}；

### 7.数组拍平

```js
function arrFlag (arr, depth) {
    if (depth === 0) {
        return arr
    }
    let res = []

    for (let i = 0; i < arr.length; i++) {
        if (Array.isArray(arr[i])) {
            res = res.concat(arrFlag(arr[i], depth - 1))
            depth + 1
        } else {
            res.push(arr[i])
        }
    }

    return res
}

let arr = [1, 2, [3, 4, [5, 6, [7, 8]]]]
console.log(arrFlag(arr, 2))
```

#### 不用遍历

> JSON.stringify 正则处理

### 8. lighthouse实现原理

#### 分步执行分析

- 页面加载：模拟用户在浏览器打开页面，监控资源加载的情况；
- 性能指标测量：页面加载时，记录性能指标，如首字节事件TTFB、首次内容绘制FCP、最大内容绘制LCP等；
- 交互性评估：检测响应时间TTI、输入延迟等；
- 可访问性检查：静态分析检查网页代码，确保符合网页标准，如SEO、可访问性、移动友好型；

### 9.css沙箱如何实现

#### CSS命名空间

> 通过使用特定的命名规范或前缀来区分不同的组件或模块，确保样式只应用于特定部分。

```css
.componentA__title {
  color: blue;
}

.componentB__title {
  color: red;
}

```

#### CSS模块化

> CSS Modules 是一种通过打包工具（如 Webpack）实现的样式模块化方案。每个模块的样式都被编译成局部作用域的样式，避免全局污染。
>
> 在编译过程中，CSS Modules 会将 `.button` 类名编译成一个唯一的类名（如 `.button_abc123`），从而实现局部作用域的样式隔离。每个组件的样式都是独立的，互不干扰。

#### Scoped CSS

> <style scoped> 会通过自动为组件内的 HTML 标签和 CSS 规则添加独特的属性选择器（如 data-v-xxxx），从而确保样式仅在当前组件内生效。

### 10.script标签放在head和body的区别？

#### 放在head

> 执行时机：页面渲染之前，解析到script标签会立即执行脚本，阻塞后面的页面渲染，知道脚本执行完毕；
>
> 影响：脚本执行慢，导致白屏时间过长；

#### 放在body底部

> 执行时机：页面渲染完毕，脚本不会阻塞页面渲染；
>
> 影响：白屏时间短，视觉内容发先加载，用户体验好。对于需要操作DOm的脚本好；

#### defer/async

> - defer：让head里的scipt即使加载完毕也要等待页面渲染完成再执行，不阻塞页面；
> - async：脚本会异步下载并尽快执行，不会阻塞页面，但执行顺序不一致；

### 11.https加密流程

#### 加密流程

1. 客户端发起https请求，此时进行握手流程；
2. 服务器返回证书，包含服务器公钥的数字证书，通常由CA签署包含以下信息：服务器公钥、证书有效期、证书颁发机构、服务器域名信息；
3. 客户端验证证书：检查是否可信、是否过期、域名是否匹配；
4. 客户端生成对话的密钥，用于对称加密；使用服务器公钥加密对话密钥；
5. 服务器接收后，使用会话密钥加密数据传输；

#### 数字证书

- 公钥
- 颁发者信息
- 有效期
- 签名：用于保证完整性和真实性；

### 12.package.json文件里有什么？

- name 项目名称
- version 版本号
- description 描述信息
- main 主文件入口
- script 运行脚本 启动项目、运行测试等
- dependencies 项目运行时依赖包及版本
- devDependencies 仅在项目开发时使用的包：测试框架、构建工具

### 13.防抖

```js
function debounce (fn, delay) {
    let timer
    return function (...args) {
        if (timer) {
            clearTimeout(timer)
        }
        timer = setTimeout(() => {
            fn.apply(this, args)
        }, delay)
    }
}
```

### 14.节流

```js
function throttle (fn, delay) {
    let timer
    return function (...args) {
        if (!timer) {
            timer = setTimeout(() => {
                fn.apply(this, args)
                timer = null
            }, delay)
        }
    }
}
```

### 15.事件循环

#### async、await

> async和await是基于Promise的语法糖，await会暂停async函数的执行，等待Promise解决或拒绝。当遇到await时，js会将后续代码放入微任务队列等待执行；

async会自动返回一个Promise；

await会暂停async的执行，等待后面的Promise resolve；期间，事件循环会继续处理其他任务，直到Promise resolve；

#### 宏任务

- setTimeout 
- setInterval
- requestAnimationFrame 用于下一帧渲染时调用某个函数
- IO操作、DOM事件

#### 微任务

- promise.then
- MutationObserver 监听DOM变动的API
- queueMicrotask 直接将任务放入微任务队列

为什么微任务优于宏任务？

> 为了确保更快速的处理异步回调，保证任务的及时执行，避免饥饿问题。

```js
console.log('Start')

setTimeout(() => {
    console.log('Timeout')
}, 0)

async function foo () {
    console.log('Inside foo')
    await Promise.resolve()
    console.log('After await')
}

foo()

Promise.resolve().then(() => {
    console.log('Promise then')
})

console.log('End')


Start
Inside foo
End
After await
Promise then
Timeout
```

### 16.http options请求

> Options方法用于浏览器跨院共享场景下进行预检请求。当网页想不同源发起跨域Http请求时，浏览器自动发送一个OPTIONS请求，确认目标服务器是否允许该实际请求。
>
> 仅支持POST、PUT、DELETE等非简单请求，或自定义请求头；
>
> 可用于查询服务器支持哪些Http方法；
>
> Get请求是否需要预检取决于请求的具体内日那个，简单Get请求无需预检，但是，如果GET请求：
>
> - 包含非标准请求头Authorization、自定义头部；
> - 使用非标准内容类型application/json之外的类型；
> - 使用了credentials选项发送带凭据的请求；
>
> 那么浏览器也可能会发出options预检请求；

### 17. vue2、3双向绑定的区别

#### vue2

> vue2通过Object.defineProperty实现，通过getter和setter来拦截对数据的访问和修改，从而实现响应式数据更新；

- 数据劫持：通过Object.defineProperty对对象的每个属性进行劫持，访问属性触发getter、修改触发setter；
- 依赖收集：getter中，将依赖该数据的组件或元素添加到一个订阅者列表中；
- 派发更新：setter触发后，vue通知所有依赖订阅者，这些依赖重新计算或更新，最终触发视图更新；

局限性：

- 深度遍历：vue2必须递归遍历对象的每一个属性来劫持，因此嵌套过深的对象，性能较差；
- 数组更新问题：vue2无法直接监听数组的索引变化或长度变化，必须使用vue.set或Array.prototype.splice保证响应式；
- 对象属性问题
- 性能瓶颈：由于 `Object.defineProperty` 是针对每个属性定义 getter 和 setter，劫持复杂对象的性能消耗较大。

#### vue3

> vue3基于Proxy对象实现响应式；

- Proxy劫持：ES6新增的Proxy可以代理整个对象，不必像vue2对每个对象属性进行劫持。这意味着可以直接监听对象本身的变化，而无需递归遍历所有属性。Proxy能拦截所有对象操作，包括属性的读取、设置、删除等；
- 依赖收集与更新：任然采用发布订阅模式；
- 懒加载：vue3的响应式系统是懒加载的，当访问某个属性的时候，才会对该属性进行依赖收集和响应式处理，避免不必要的开销；
- 优化数组：vue3能够原生监听数组的索引变化和长度变化，基于Proxy能够代理js对象的基本方法实现。

### 18.一个自定义组件如何实现v-model

```vue
<template>
  <input
    :value="modelValue"
    @input="updateValue($event.target.value)"
  />
</template>

<script>
export default {
  name: 'MyInput',
  props: {
    modelValue: {
      type: String,
      default: ''
    }
  },
  methods: {
    updateValue(value) {
      // 触发 update:modelValue 事件，将新值传递给父组件
      this.$emit('update:modelValue', value);
    }
  }
}
</script>

```

### 19.v-model（双向绑定）实现原理

> 数据模型到视图：数据模型发生变化，通过响应式原理自动更新视图；
>
> 视图到数据模型：视图数据被用户修改，通过监听事件和数据更新事件；

```html
<input :value="message" @input="message = $event.target.value" />

```

### 20.vue修饰符

#### .lazy

> 将输入事件从input改成change，延迟数据更新

#### .number

#### .trim



### 21.vue组件间通信

- 父子通信：props $emit
- 兄弟组件：通过公共父组件中专；
- 非父子组件：provide/inject
- vuex
- 插槽 默认，具名，作用域插槽（同时使用父组件域内和子组件域内的数据。要做到这一点，我们需要一种方法来让子组件在渲染时将一部分数据提供给插槽）

### 22.撕实现一个sum(1)(2)(3).value()=6

```js
function sum (num) {
    let total = num

    function innerSum (nextNum) {
        total += nextNum
        return innerSum
    }

    innerSum.value = function () {
        return total
    }

    return innerSum
}

// 使用示例
console.log(sum(1)(2)(3)(4).value()) // 输出 6

```

### 23.WeakMap、WeakSet

> 主要用于内存管理和性能优化；

#### WeakMap

- 键是弱引用，WeakMap的键是对象，并且是弱引用，如果没有其他对该对象的引用，垃圾回收器可以自动回收这个对象；
- 不支持遍历，无法使用forEach 或 for of
- 只接受对象为键，不能是原始类型；

#### WeakSet

- **值是弱引用**：`WeakSet` 的值是对象，并且是弱引用。这意味着，存储在 `WeakSet` 中的对象也可以被垃圾回收。
- 不支持遍历，无法使用forEach 或 for of
- 只接受对象为键，不能是原始类型；

### 24.JS垃圾回收机制

> 主要目标是回收不再被引用的对象，释放内存；

#### 标记清除

##### 标记阶段

- 从根对象如全局对象、活动函数的局部变量等开始，遍历所有可访问的对象，并标记他们为活得；
- 遍历过程中无法到达的对象不会被标记；

#### 清除阶段

- 遍历所有对象，删除未被标记的对象；

#### 引用计数

- 每当一个新的引用指向对象时，该对象的引用计数增加；
- 当一个引用被删除或超出作用域时，该对象的引用计数减少；
- 当计数为0时，回收对象；

##### 问题

- 无法处理循环引用

##### 建议

- 减少闭包的使用；
- 减少循环引用，使用WeakMap或WeakSet

### 25.什么是闭包？

> 闭包（Closure）是 JavaScript 中一个非常重要的概念，它指的是一个函数与其词法环境的结合。简单来说，闭包是一个能够“记住”并访问其外部作用域变量的函数，即使外部作用域已经执行完毕。



> 
>
> - 几大块、fiber算法、运行机制
> - diff算法、vdom
> - 社区库跑起来、运行起来

### 26.vue3新特性

- composition API 全新方式组织和复用逻辑，增强代码可读性和可维护性；
- Teleport 允许将组件渲染位置传送到DOM中的其他位置，适用于拟态框；
- Fragments 支持多个根节点，允许组件返回多个顶层元素，而不必包裹在一个根元素内；
- 重写虚拟DOM；
- TS支持；
- 新生命周期钩子 onBeforeMount、onBeforeUpdate、onBeforeUnmount；

#### 重写虚拟DOM

- 编译时优化：编译阶段将模板中的静态部分提取出来，生成高效渲染代码；静态节点不会在每次更新时重新渲染，避免了不必要的虚拟 DOM diff 操作，从而提升了性能。
- Block Tree：组件更新时不再需要对整个组件树做完整的diff操作，通过Block Tree只更新变化的部分；
- Patch Flags：使用Flag标记虚拟DOM的变动区域，跳过不必要的更新；
- 双端Diff算法：头尾双向扫描；
- 使用Proxy代替



### 27.vue指令模板如何实现的？

> 带有`v-`前缀的特殊属性，在模板中对DOM元素进行特定的操作；

#### 指令定义

> vue指令通过JS对象定义，借助生命周期钩子执行特定操作；

#### 模板编译

> vue组件模板被编译时，vue会解析指令并将其转换为对应的渲染函数。这个过程会将指令信息和操作存储在虚拟DOM；
>
> - **解析指令**：Vue 会扫描模板，识别出所有的指令，并提取出指令名及其参数。
> - **生成渲染函数**：每个指令都会被转化为一段 JavaScript 代码，用于描述如何在虚拟 DOM 中进行操作。



### 28.vuex实现原理

> 集中式存储来管理所有组件的状态，使得组件的管理更加规范和可预测；

- store
- state
- getters
- mutations：同步更新状态，修改state的唯一途径
- actions：异步操作，最后调用mutation更新；
- modules：支持将store切分成多个模块；

#### 实现原理

> vue将所有的状态放在一个单一的store中，组件通过mapState或mapGetters等辅助函数访问。状态是响应式的；



### 29.redux实现原理



### 30.vue3有哪些性能优化

- 重写虚拟DOM：双端比较算法diff、Patch Flags
- 编译优化：静态节点提升为常量，减少渲染；
- Tree-shaking
- Proxy
- 懒加载和代码分割；
- 计算属性优化：只在依赖的数据发生变化时重新计算，而不会在每次渲染时都重新执行，减少了计算开销。
- 更好的内存管理：WeakMap、WeakSet

### 31.http1 2 3

#### Http 0.9 

- HTTP/0.9 的设计非常简单，只支持单一的请求方法：`GET`。
- 只允许请求单个文档，无法处理复杂的请求。
- 协议是无状态的，每个请求之间没有上下文，服务器不会记住之前的请求。
- HTTP/0.9 主要用于传输纯文本文件，没有对二进制文件或其他 MIME 类型的支持。

#### Http1.1

- **持久连接**：允许在一个 TCP 连接上发送多个请求和响应，减少了建立连接的开销。
- **管道化**：可以在前一个请求未完成时发送多个请求，但并不广泛使用，因为响应顺序不一致。
- **缓存控制**：引入了更多的缓存机制，通过 HTTP 头部字段来控制缓存行为。
- **分块传输**：支持将响应分块传输，允许在未知总内容长度时传输数据。

### Http 2

- **二进制协议**：HTTP/2 使用二进制格式而非文本格式，提高了解析效率和减少了传输大小。
- **多路复用**：允许在同一连接上并行发送多个请求和响应，消除了队头阻塞（Head-of-line blocking）问题。
- **优先级**：支持请求的优先级，可以为更重要的请求分配更多资源。
- **服务器推送**：服务器可以主动向客户端推送资源，而无需等待客户端请求。
- **有效的压缩**：使用 HPACK 算法压缩 HTTP 头，减少传输的冗余。

### Http 3

- **基于 UDP**：HTTP/3 使用 QUIC 协议，基于 UDP 而非 TCP，能够减少连接建立和丢包重传的延迟。
- **更快的连接建立**：通过 0-RTT 和 1-RTT 的连接建立方式，减少了延迟。
- **多路复用**：避免了队头阻塞，不同于 HTTP/2 的 TCP 多路复用，QUIC 允许在不同流中独立处理数据。
- **内建加密**：QUIC 协议内置 TLS 加密，简化了安全配置，提高了安全性。



### 32.XSS、CSRF

#### XSS跨站脚本攻击

> 攻击者能够在用户的浏览器注入恶意脚本、窃取用户信息；
>
> - 存储型：脚本存储在服务器数据库；
> - 反射型：脚本通过URL参数或表单提交即时返回，未存储在服务器；
> - DOM型：通过修改页面DOM结构来执行恶意脚本；
>
> 防范方法：
>
> - 输入验证：过滤用户输入，避免恶意注入；
> - 输出编码：对动态生成的HTML内容编码；
> - 使用CSP
> - 避免使用InnerHTML

#### CSRF跨站请求伪造

> 攻击者诱导已认证的用户在不知情的情况下向 Web 应用发送请求，从而执行恶意操作（如转账、修改设置等）。
>
> 原理：
>
> - 用户在网站 A 上登录并获取会话。
> - 攻击者构造一个指向网站 A 的请求，并通过电子邮件、社交媒体等诱导用户点击该链接。
> - 用户在登录状态下点击链接，浏览器自动携带用户的会话 Cookie，执行请求，导致未授权的操作。
>
> 防范方法：
>
> - CSRF Token：随机tolen验证该token是否正确；
> - Samesite Cookie：使用SameSite来限制COokie的发送策略，减少跨站请求的可能性；
> - Referer检查：验证请求的来源，确保请求合法；
> - 用户验证：对敏感操作进行额外用户验证，如二次确认或输入密码；



### 33.v8垃圾回收机制



### 34.webpack用过哪些功能？



### 35.loader和module有什么区别？

#### Loader

> Loader是一种转换器，用于对模块进行预处理，支持将不同类型的文件，如CSS、图片、Markdown等转换为JS模块；
>
> 例如：
>
> - Babel Loader 将ES6+的迪马转换为ES5
> - CSS Loader 将CSS文件转换为JS对象
> - style-loader 将css以<style>标签的形式注入到DOM
> - file-loader 处理文件导入，将文件输出到指定目录，并返回URL
> - vue-loader 处理vue单文件组件；

```js
module: {
  rules: [
    {
      test: /\.js$/,
      use: 'babel-loader'
    },
    {
      test: /\.css$/,
      use: ['style-loader', 'css-loader']
    }
  ]
}

```

#### Module

> Module是代码的基本单位，表示一个独立的代码块，通常可以导入和到处；
>
> JS中有很多实现，如CommonJS、AMD、ES6 Modules

#### Plugin

> Plugin是可以被集成到工具中的模块，允许开发者在工具的生命周期中添加特定的功能或行为。
>
> - **扩展功能**：为工具添加新的功能，比如代码压缩、代码分割、热更新等。
> - **自定义构建过程**：允许开发者在构建过程中自定义某些步骤，执行特定的任务，如在构建前后进行某些操作。
> - **优化性能**：通过分析和优化资源的使用，提升构建和运行的性能。

```js
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  // ...
  plugins: [
    new HtmlWebpackPlugin({
      template: './src/index.html',
      filename: 'index.html'
    })
  ]
};

```

> - HhmlWebpackPlugin 生成HTML文件，自动引入打包后资源
> - MiniCssExtractPlugin 将CSS提取为单独文件；
> - TerserPlugin 压缩优化JS代码
> - HotModuleReplacementPlugin 热更新











