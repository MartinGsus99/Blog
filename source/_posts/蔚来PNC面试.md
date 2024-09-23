### 1.VUE2和VUE3的区别

- 双向数据绑定的原理不同

> v2的双向数据绑定是通过ES5的API Object.defineProperty()来对数据进行劫持，集合发布订阅模式实现
>
> v3中使用ES6的Proxy API实现
>
> 使用Proxy的优势有：
>
> defineProperty只能监听属性，不能对整个对象进行监听
>
> 可以省去for in，闭包等内容来提升效率（直接绑定整个对象即可）
>
> 可以监听数组，不用再单独的对数组做特异性操作，Vue3可以检测到数组内部数据的变化

- v2不支持碎片，v3支持碎片（有多个根节点）
- API类型不同

> ​	v2使用选项类型APPI，在代码里分割不同属性，data,computed,method
>
> ​	v3使用合成型api,新的合成型api 能让我们使用方法来分割，相比于旧的api 使用属性来分组，这样代码会更加简便和整洁。

- 定义数据变量和方法不同

> Vue2是把数据放到了data 中，在 Vue2中 定义数据变量是data(){},创建的方法要在method:{}
>
> Vue3 就需要使用一个新的setup()方法，此方法在组件初始化构造的时候触发。使用以下三个步骤来建立反应性数据：
>
> 1）从vue 引入 reactive；
>
> 2）使用 reactive ()方法来声明数据为响应性数据；
>
> 3） 使用setup()方法来返回我们的响应性数据，从而template 可以获取这些响应性数据。

- 生命周期钩子函数不同

> Vue2 中的生命周期：beforeCreate 组件创建之前；created 组建创建之后；beforeMount 组件挂载到页面之前执行；Mounted 组件挂载到页面之后执行，beforeUpdate 组件更新之前；updated组件更新之后
>
> Vue3 中的生命周期：setup 开始创建组件；onBeforeMount 组件挂载到页面之前执行；onMounted 组件挂载到页面之后执行；onBeforeUpdate 组件更新之前；onUpdated 组件更新之后；
>
> 而且 Vue3 生命周期在调用前需要先进行引入。除了这些钩子函数外，Vue3 还增加了 onRenderTracked 和onRenderTriggered 函数。

- 父子传参不同

> Vue2 父传子，用props ；子传父用事件Emitting Events。
>
> 在Vue2 中，会调用this$emit 然后传入事件名和对象。
>
> Vue3 父传子，用props; 子传父用Emitting Events 。
>
> 在Vue3 中的setup()中的第一参数content 对象中就有 emit,那么我们只要在setup()接收第二个参数中使用分解对象法取出emit 就可以在setup 方法中随意使用了。



### 2.vue响应式原理

> vue实现数据响应式，是通过数据劫持侦测数据变化，发布订阅模式进行依赖收集与视图更新，换句话说是Observe，Watcher以及Compile三者相互配合
>
> Observe实现数据劫持，递归给对象属性，绑定setter和getter函数，属性改变时，通知订阅者
> Compile解析模板，把模板中变量换成数据，绑定更新函数，添加订阅者，收到通知就执行更新函数
> Watcher作为Observe和Compile中间的桥梁，订阅Observe属性变化的消息，触发Compile更新函数

### 3.vue组件通信的方式 √

- 父传子：props
- 子传父：this.$emit
- 祖先provide传递，子孙组件inject接收
- 父组件通过`this.$refs`获取子组件数据
- 子组件通过`this.$parent`获取父组件数据
- `$attrs`
- `$listeners`
- 事件总线`event-bus`
- slot作用域传值

```vue
<template>
  <div>
    <aaaChildren>
      <template #default="scope">
        <h1>{{ scope.info.age }}</h1>
      </template>
      <template #text="scope">
        <h2>{{ scope.data }}</h2>
      </template>
    </aaaChildren>
  </div>
</template>

<script>
import aaaChildren from './children/index.vue'

export default {
  name:'aaa',
  components:{
    aaaChildren
  }
}
</script>

```

```vue
<template>
  <div class="aaa_children">
    <slot :info="user"></slot>
    <slot name="text" :data="name"></slot>
  </div>
</template>

<script>
  export default {
    name:'aaaChildren',
    props:[],
    components:{},
    data(){
      return {
        user: {
          age: 18,
          name: '小明'
        },
        name: '卡卡罗特'
      }
    }
  }
</script>

```



### 4.盒模型，标准盒模型、怪异盒模型

W3C标准盒子模型由：content、margin、padding、border组成；

##### 区别：总宽度的计算公式不一样

标准盒模型总宽度=width+margin（左右）+padding（左右）+border（左右）；

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201224112703410.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDgwODQ4Mw==,size_16,color_FFFFFF,t_70#pic_center)

怪异盒模型总宽度=width+margin（左右）（width已经包含padding和border的值）；

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201224113002950.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDgwODQ4Mw==,size_16,color_FFFFFF,t_70#pic_center)

### 5.JS事件机制

> html中的标签都是相互嵌套的，document是最外面的大盒子。当你单击一个div时，同时你也单击了div的父元素，甚至整个页面。那div的单击事件如何执行？这就涉及到 JS中 DOM[事件流](https://so.csdn.net/so/search?q=事件流&spm=1001.2101.3001.7020) 的相关知识点了

- 事件流：从页面中接收事件的顺序，事件发生时在元素节点按照一定的顺序执行就叫做DOM事件流。
- 事件冒泡：事件的逐级向上传导的过程，当后代元素上的事件被触发时，其祖先元素的相同事件也会被触发。
- 事件捕获：从最外层开始，然后一层一层接收事件并响应，也就是捕获事件流。最终由w3c统一了最后标准，先捕获在冒泡，现代的浏览器都遵循此标准。

### 6.事件冒泡、怎样阻止冒泡

- 事件对象：事件发生后相关的一系列信息数据都放在对象里；
- 事件对象的使用

> 1.标准浏览器是浏览器给方法传递的参数，只需要定义形参e就可以获取到
>
> 2.IE6-8中，浏览器不会给方法传递参数，需要的话，到window;event中获取

- 阻止事件的默认行为

> e.preventDefault()

- 阻止冒泡

> e.stopPropagation()
>
> window.event.cancelBubble=true

### 7.position属性的用法

- static:包含此属性的元素遵循常规流，即块级元素占一行，行内元素依次排列同一行，当超过父布局最大宽度后自动换行。


- relative：在不设置`top,right,bottom,left`这些属性时，与static无区别，加上后会相对于自身在常规流中的位置进行定位；
- absolute：相对定位`relative`没有脱离文档流，但是`absolute`脱离了文档流；ralative相对于自身在常规流中的位置进行偏移定位，而`absolute`相对于离自身最近的定位祖先元素进行定位；拥有相对定位属性值的祖先元素可以充当拥有绝对定位属性值的子孙元素的定位祖先元素。如果子孙元素没有定位祖先元素，会一直回溯到body元素，使用body元素充当自己的定位祖先元素。

```css
    body {
      border: 1px solid black;
      padding: 12px;
      position: relative;
    }

    .parent {
      width: 300px;
      height: 300px;
      background-color: #ff0;
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
    }
```

https://blog.csdn.net/qq_16121469/article/details/122357306

- fixed：fixed相对于窗口进行偏移定位；absolute相对于祖先元素进行偏移定位；

```css
    body {
      border: 1px solid black;
      padding: 12px;
      height: 1000px;
    }

    .center {
      position: fixed;
      width: 100px;
      height: 100px;
      background-color: #f00;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
    }
```

- sticky：类似`relative`和`fixed`的结合，若不设置`top,right,bottom,left`则和static一致；设置了则存在两种状态，以top：10px;为例，若元素相对于窗口顶部距离大于2.px，则处于类似relative相对定位的状态；若小于等于10px，则立马切换到fixed的状态；

```css
       body {
                border: 1px solid black;
                padding: 32px;
                margin: 32px;
                height: 1000px;
            }
            div {
                width: 100px;
                height: 100px;
                position: relative;
            }
            #div1 {
                background-color: red;
            }
            #div2 {
                background-color: black;
                position: sticky;
                top: 100px;
                left: 100px;
                padding: 0;
                margin: 0;
            }
            #div3 {
                background-color: gray;
            }
```



### 8.Felx布局，Flex：1

[flex](https://so.csdn.net/so/search?q=flex&spm=1001.2101.3001.7020)：1即为flex-grow：1，经常用作自适应布局，将父容器的display：flex，侧边栏大小固定后，将内容区flex：1，内容区则会自动放大占满剩余空间。

flex属性 是 flex-grow、flex-shrink、flex-basis三个属性的缩写。

### 9.leetcode 738



### 10.水平垂直居中

- `margin:auto`

  ```css
      .box1 {
        background-color: #bfa;
        width: 500px;
        height: 500px;
        position: relative;
      }
  
      .box2 {
        background-color: red;
        width: 100px;
        height: 100px;
        position: absolute;
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;
        margin: auto;
      }
  ```

  

- `display:table-cell`

```css
     .box1{
            background-color: #bfa;
            width: 500px;
            height: 500px;
            display: table-cell;
            vertical-align: middle;
            text-align: center;
        }
        .box2{
            background-color: red;
            width: 100px;
            height: 100px;
            display: inline-block;
        }
```

- `flex`方法

```css
   .box1 {
      background-color: #bfa;
      width: 500px;
      height: 500px;
      display: flex;

      justify-content: center;
      align-items: center;

    }

    .box2 {
      background-color: red;
      width: 100px;
      height: 100px;

    }
```

- 手动计算

```css
    .box1 {
      background-color: #bfa;
      width: 500px;
      height: 500px;

    }

    .box2 {
      background-color: red;
      width: 100px;
      height: 100px;
      margin-left: 200px;
      margin-top: 200px;

    }

    .box1::before {
      content: "";
      display: table;
    }
```

- `translate`

```css
    .box1 {
      background-color: #bfa;
      width: 500px;
      height: 500px;
      position: relative;
    }

    .box2 {
      background-color: red;
      width: 100px;
      height: 100px;
      position: absolute;
      top: 50%;
      left: 50%;

      transform: translate(-50%, -50%);
    }
```



### 11.隐藏元素的方法

- overflow:hiddeb;
- display:none;
- opacity:0;
- position:absloute; top:-9999px;left:-9999px;
- **visibility:hidden**,元素消失后，位置空间保留，只会导致重绘不会导致重排；
- 设置盒模型属性为0



### 12.重排重绘

> ***\*重绘就是重新绘制（repaint）：\****是在一个元素的外观被改变所触发的浏览器行为，浏览器会根据元素的新属性重新绘制，使元素呈现新的外观。
>
> ***\*重排就是重新排列（reflow）：\****当渲染树的一部分必须更新并且节点的尺寸发生了变化，浏览器会使渲染树中受到影响的部分失效，并重新构造渲染树。

改变元素几何信息（大小和位置），都会引起：


        元素位置改变，或者使用动画
    
        元素的尺寸改变（外边距、内边距、边框厚度、宽高等几何属性）
    
        内容改变（例如：文本改变或图片被另一个不同尺寸的图片替代、在input框输入内容）
    
        浏览器窗口尺寸改变（resize事件发生时）
    
        页面渲染初始化
    
        设置style属性的值
    
        计算offsetWidth、offsetHeight、offsetTop和offsetLeft等布局信息
    
        激活CSS伪类（如 :hover）
    
        查询某些属性或调用某些方法（如：getComputedStyle()、getBoundingClientRect()）
减少重排盒重绘：

        避免一条一条的修改DOM的样式，可以直接修改DOM的className
    
        避免把DOM结点的属性值放在一个循环里当成循环里的变量
    
        给动画的HTML元件使用fixed或absolute的position，那么修改他们的css是不会重排
    
        避免在大量元素上使用:hover
    
        分离读写操作
    
        避免使用Table布局
    
        避免设置多层内联样式
    
        避免在布局信息改变时查询布局信息
    
        当需要对DOM元素进行一系列的操作时，可以先使元素脱离文档流，再对其进行一些列操作，然后再把元素带回文档中


### 13.JS数据类型

- 基本数据类型（原始值）：

  Number（数值，包含NaN）
  String（字符串）
  Boolean（布尔值）
  Undefined（未定义/未初始化）
  Null（空对象）
  Symbol（独一无二的值，ES6 新增）

  > 注意， Symbol() 函数的参数只有描述作用，即使两个 Symbol() 函数的参数相同 ，它们的返回值也是不相等的。

  BigInt （大整数，能够表示超过 Number 类型大小限制的整数，ES 2020新增）

- 引用数据类型（引用值）：

  Object（对象。Array/数组 和 function/函数 也属于对象的一种）
  

### 14.闭包，举例

> 当一个函数定义在另一个函数内部，并且内部函数引用了外部函数作用域中的变量，就会创建一个闭包。
>
> 当内部函数从外部函数返回时，它会保留对外部函数作用于的引用，即使在外部函数执行完毕后，仍然可以继续访问这些变量。
>
> 通常用于从外部父函数检索值，可以利用闭包来检索已经超出范围的死值，还可以用于保护某些变量或者函数。
>
> 闭包对于创建私有变量和函数、实现部分函数应用以及在异步代码中保留状态非常有用。

```js
const makeCounter=()=>{
    let count =0
    
    return ()=>{
        count++
        console.log(count)
    }
}

let counter=new MakeCounter();
counter()		//1
counter()		//2
counter()		//3
```

由于counter并么有暴露在返回对象的外部，它是一个只能通过makeCounter（）进行访问或者修改的私有变量；

```js
function add(x){
    return function(y){
        return x+y
    }
}

let add5=add(5)
console.log(add5(3))   //5+3
```

在这个例子中，add()函数返回了另一个接受单个参数的函数，该函数返回参数与外部函数作用域中x值的总和。

### 15.setTime（cb,0)



### 16.事件循环

> Event Loop即事件循环，JS为单线程语言，同一时间内只能做一件事情，但是并不意味着单线程就是阻塞，实现单线程非阻塞的方法就是事件循环；
>
> 同步任务：立即执行的任务，一般进入主线程执行；
>
> 异步任务：异步执行的任务，比如ajax网络请求，setTimeout定时函数等；

> 同步任务进入主线程，异步任务进入任务队列，会在分为宏任务（宏任务队列）和微任务（微任务队列）；
>
> 当主线程任务为空时，检查微任务的任务队列，全部执行完后执行宏任务队列；
>
> 每执行完一个宏任务就清空一次微任务队列，此过程不断重复，这就是Event Loop

- 宏任务

  > - setInterval()
  > - setTimeout()
  > - setImmediate()
  > - ajax
  > - 事件绑定

- 微任务

  > - new Promise()后的then与catch函数
  > - new MutaionObserver()
  > - process.nextTick（Nodejs）

```js
setTimeout(() => {
    console.log('异步1任务time1');
    new Promise(function (resolve, reject) {
        console.log('异步1宏任务promise');
        setTimeout(() => {
            console.log('异步1任务time2');
        }, 0);
        resolve();
    }).then(function () {
        console.log('异步1微任务then')
    })
}, 0);

console.log('主线程宏任务');

setTimeout(() => {
    console.log('异步2任务time2');
}, 0);

new Promise(function (resolve, reject) {
    console.log('宏任务promise');
    // reject();
    resolve();
}).then(function () {
    console.log('微任务then')
}).catch(function () {
    console.log('微任务catch')
})

console.log('主线程宏任务2');
```



```node
主线程宏任务
宏任务promise
主线程宏任务2
微任务then
异步1任务time1
异步1宏任务promise
异步1微任务then
异步2任务time2
异步1任务time2
```



### 17.== 、===的区别

- `==`先检查是否为相同数据类型，如果不同，则进行一次数据类型转换在进行比较；
- `===`如果类型不同则直接返回false，不进行类型转换；

### 18.输入url之后详细的过程

https://blog.csdn.net/weiCong_Ling/article/details/130638983

1. 浏览器先查找当前URL的DNS缓存记录，如果存在缓存，则直接显示，没有则进行下一步；
2. DNS域名解析：对域名进行解析，获得IP地址；如果本地DNS缓存有则直接获取，没有就向DNS服务器请求，DNS查询的方式有迭代和递归两种。
3. 查询到IP地址后进行TCP的建立，TCP会进行三次握手，本地客户端向服务器发送SYN包建立链接的请求，请求通过后服务器返回ACK+SYN包给客户端，客户端获得ACK包后在发送ACK包完成三次握手，握手完成后进行数据的传递；
4. 发送http请求报文，请求包括请求方法，URL，协议版本等
5. 服务器返回response响应，断开TCP链接，四次握手，客户端发送FIN包，服务器发送ACK包，再发送FIN+ACK包结束数据传送；
6. 本地浏览器开始对http请求获得资源，比如JS，html等进行解析，先解析html，生成DOM树，解析css生成css规则树，然后进行渲染，GPU绘制，挂载完成；
7. 解析html时遇到图片样式js文件会进行下载；

> 回流：当某个元素的尺寸带线啊哦或者位置信息发生改变时，触发回流，对元素的大小和位置进行重新计算；
>
> 重绘：当某个元素的背景颜色，文字颜色或者其他不影响周围或者内部布局的属性发生改变时触发重绘；
>
> 回流一定会重绘，重绘不一定回流；

### 19.项目困难问题

- 权限控制问题：需要根据角色来显示不同的页面；

> 通过获取角色显示角色的权限页面动态加载；
>
> 1.将常规权限页面放在一起，动态权限放在一起；
>
> 2.登陆时获取其权限，

- 需要根据数据绘制知识图谱

> 学习G6组件库，绘制知识图谱

- 使用的Elment组件无法自定义样式

> 通过v-deep

- 权限路由动态加载必须登录后刷新一次才能显示

> 改成退出登陆后刷新一次浏览器

### 20.css实现三角形

> https://mp.weixin.qq.com/s?__biz=MzU2MTIyNDUwMA==&mid=2247493438&idx=1&sn=86e6cb82dc1e7bf591e38dcc81105552&chksm=fc7ea965cb0920736dcfb8876892c68e5b667db77728aa1b18755cdefce1d82c441c9bafcd48

```css
.triangle {
 border-style: solid;
  border-color: transparent;
  border-width: 50px 0 50px 50px;
  border-left-color: skyblue;
}
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/EO58xpw5UMMnSI8IngMJeUCXvxMdFtmGBwIYK5ZCmIhicvPS0jTeqZic7kTMGHB6kjJTuA908awibG0pib2LxAbGfA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



```css
.triangle {
    width: 0;
    height: 0;
    border-top: 100px solid skyblue;
    border-right: 100px solid transparent;
}
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/EO58xpw5UMMnSI8IngMJeUCXvxMdFtmGrXwwoUHHQFIPT4VicAgV27R6hR5JVcecIUXAvAl7uBpQianD7JdPwD2Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

### 21.http和https的区别 √

       1、https协议需要到CA申请证书，一般免费证书较少，因而需要一定费用。
       2、http是超文本传输协议，信息是明文传输，https则是具有安全性的ssl/tls加密传输协议。
       3、http和https使用的是完全不同的连接方式，用的端口也不一样，前者是80，后者是443。
       4、http的连接很简单，是无状态的；HTTPS协议是由SSL/TLS+HTTP协议构建的可进行加密传输、身份认证的网络协议，比http协议安全。
### 23.深浅拷贝

> 浅拷贝时单层拷贝
>
> 会在栈中开辟另一块空间，并将被拷贝对象的栈内存数据完全拷贝到该块空间中，即基本数据类型的值会被完全拷贝，而引用类型的值则是拷贝了“指向堆内存的地址”。

浅拷贝方法：

- Object.assign()
- 扩展运算符(…)
- Array.concat()
- Array.slice()

> 深拷贝是拷贝多层，每一级别的数据都会拷贝出来

深拷贝方法：

- JSON.parse(JSON.stringify(obj))
- 递归方法
- 函数库 lodash

1. 浅拷贝主要用于你需要拷贝的对象的数据结构只有基础数据类型，并且你不想改变原数据类型或者需要对比操作前后的数据。
2. 深拷贝主要用于你想操作该数据，但是又不想影响到原数据的时候，就可以进行深拷贝。

### 24.cookie、sessionStorage、localStorage的区别  √

#### （1）cookie

> cooie是客户端与服务器端进行会话使用的一个能够在浏览器本地化存储的技术；
>
> cookie以字符串的形式存储，以key=value的形式存储，以；分割

http是无状态协议，为了让服务器能够识别客户端，客户端使用cookie，服务端使用session。当浏览器首次发送请求时，服务端会产生唯一的编号sessionId,响应时，把sessionId发送给浏览器端，浏览器端把sessionId保存到cookie中，下次请求时，带上sessionId，服务器根据不同的sessionId来区分不同客户端；

- 会话状态管理（用户登录状态，购物车）
- 个性化设置（保存用户设置的样式等）
- 浏览器行为跟踪（分析用户行为）

cookie可以在F12 Application中看到

#### （2）webStorage

> webstorage是HTML5新增的存储数据的方案，比使用cookie更加直观，提供了访问特定域名下的会话存储或者本地存储的功能；如果是会话存储则使用sessionStorage；如果是本地存储，使用localStorage；

- LocalStorage

> localStorage的生命周期是永久，除非手动清除，否则永远存在；存储大小是5MB，仅在客户端浏览器上存储，不参与服务器的通信；

```js
localStorage.setItem('username','Admin')
localStorage.getIte,('username')
localStorage.removeItem('username')
localStorage.clear()
```

- sessionStorage

> sessionStorage在同源的同窗口中，始终存在的数据，只要浏览器不关闭，即使是刷新或者进入另一个页面，数据任在；关闭窗口后sessionStorage就会销毁

```js
sessionStorage.setItem('sessionName','coco')
sessinStorage.getItem('sessionName')
sessinStorage.removeItem('username')
sessinStorage.clear()
```

异同：

https://blog.csdn.net/weixin_40381947/article/details/130059266

### 25.clientX，clientY 

https://blog.csdn.net/u012320487/article/details/121262745

- clientX:设置或获取鼠标指针位置相对于窗口客户区的x坐标，其中客户区域不包括窗口自身的控件和滚动条；
- clientY:设置或获取鼠标指针位置相对于窗口客户区的y坐标，其中客户区域不包括窗口自身的控件和滚动条；
- screenX：设置或获取鼠标指针位置相对于用户屏幕的x坐标；
- screenY：设置或获取鼠标指针位置相对于用户屏幕的y坐标
- x:设置或获取鼠标指针位置相对于父文档的 x 像素坐标
- y:设置或获取鼠标指针位置相对于父文档的 y 像素坐标。

![img](https://img-blog.csdnimg.cn/104b61ef1c134b62be0b2863084212f8.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBARF9qaW5nMjA=,size_20,color_FFFFFF,t_70,g_se,x_16)



### 26.什么是`vue-router`?

> vue router是vueJS的官方路由，与vuejs核心深度集成，方便构建单页面应用；

- 嵌套路由映射
- 动态路由选择
- 模块化、基于组件的路由配置
- 路由参数、查询、通配符
- 展示由 Vue.js 的过渡系统提供的过渡效果
- 细致的导航控制
- 自动激活 CSS 类的链接
- HTML5 history 模式或 hash 模式
- 可定制的滚动行为
- URL 的正确编码



### 27.



