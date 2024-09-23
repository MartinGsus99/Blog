# Vivo前端面经

### 1.怎么制定前端规范？

#### 命令规范

- camelCase（小驼峰式命名法 —— 首字母小写）
- PascalCase（大驼峰式命名法 —— 首字母大写）
- kebab-case（短横线连接式）
- Snake （下划线连接式）

##### 项目名称

全小写-短横线



##### 目录名

参照项目命名规则，有复数结构时，要采用复数命名法。例：docs、assets、components、directives、mixins、utils、views。



#### vue组件命名

##### 单文件组件名

文件扩展名为 .vue 的 single-file components (单文件组件)。单文件组件名应该始终是单词大写开头 (PascalCase)。

https://zhuanlan.zhihu.com/p/654792921



#### 代码参数名







### 2.用过Element没有？有哪些组件？



### 3.v-if和v-show的区别？

> 1、渲染方式，v-if是惰性渲染，v-show则是控制元素的显示和隐藏；
>
> 2、初始渲染开销，v-if在初始渲染时，如果条件为假，就不会渲染，可以减少开销，v-show会在初始渲染时就全部渲染；v-show则会在初始渲染时将所有元素都渲染到DOM中，只是通过CSS来控制其显示和隐藏。
>
> 3、切换开销，v-if在条件切换时会有开销，v-show只需要控制元素的显示和隐藏。

#### 使用场景

> 如果需要在条件切换频繁的情况下，可以使用v-show来避免频繁的创建和销毁组件或元素，提高性能。
>
> 如果需要在条件切换较少的情况下，可以使用v-if来在条件为假时减少不必要的渲染，节省内存。

### 4.父子组件之间传值方式

- 父传子：props
- 子传父：this.$emit
- 祖先provide传递，子孙组件inject接收
- 父组件通过`this.$refs`获取子组件数据
- 子组件通过`this.$parent`获取父组件数据
- `$attrs`
- `$listeners`
- 事件总线`event-bus`
- slot作用域传值



### 5.路由懒加载怎么实现？路由如何跳转传参？那里用到了路由传参？

#### 懒路由加载

- 箭头函数+import

```js
const List = () => import('@/components/list.vue')
const router = new VueRouter({
  routes: [
    { path: '/list', component: List }
  ]
})

```

> - 优点
>   - 语法简洁，直观易懂。
>   - 使用箭头函数和动态 `import` 实现懒加载，符合现代 JavaScript 标准。
> - 缺点
>   - 需要构建工具（如 webpack）的支持，以及启用代码分割功能。

- 箭头函数+require

```js
const router = new Router({
  routes: [
   {
     path: '/list',
     component: resolve => require(['@/components/list'], resolve)
   }
  ]
})

```

> - 优点
>   - 使用箭头函数和 `require` 实现懒加载，能够按需加载组件。
>   - 在一些旧版本的构建工具中仍然比较常见。
> - 缺点
>   - 语法相对复杂，不够直观。
>   - 依赖于旧版本的模块加载方式，可能在未来的 JavaScript 标准中被逐渐淘汰。

#### 路由传参

- 使用占位符定义动态路由

```js
const router = new VueRouter({
  routes: [
    { path: '/user/:id', component: User }
  ]
})

```

- 通过`$route.params`来获取传递过来的动态参数

```js
export default {
  created() {
    console.log(this.$route.params.id);
  }
}

```

##### Query方式

另外一种传递参数的方式是使用 Query 参数。这种方式不会改变 URL 中的路由路径，而是在路径后面以 `?` 开头加上键值对的方式传递参数。

1. 定义路由

```js
const router = new VueRouter({
  routes: [
    { path: '/user', component: User }
  ]
})

```

2. 跳转路由

```js
this.$router.push({ name: 'users', query:{ uname:'james' }})

```

3. 获取参数

```js
export default {
  created() {
    console.log(this.$route.query.uname);
  }
}

```



### 6.用过那些版本控制工具？怎么回退？怎么解决冲突？

```shell
gir reset --hard 提交版本
```



### 7.能接受加班？



### 8.实习时间？



### 9.vue2和vue3的区别？

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

### 10.vue-cli和vite的区别？

vue-cli是Vue早期推出的一款脚手架,使用webpack创建Vue项目,可以选择安装需要的各种插件,比如Vuex

#### vue-cli

> 是Vue2.0最棒的前端构建工具，是WebPack的超集
>
> Vue-cli 基于WebPack构建，配置好了打包规则
>
> 内置了热模块重载的开发服务器
>
> 有丰富的官方插件合集，站在webpack庞大的社区资源上
>
> 友好的图形化创建和管理Vue项目界面 : vue ui
>
> vue-cli在(前端)服务启动之前，要把所有代码打包成Bundle再启动服务，这也是为什么一个些大型项目 启动时，特别慢的原因。这一点在Vite做了大幅度改善。

#### vite

> Vite是Vue团队开发的新一代前端开发与构建工具，vite不是基于webpack,
>
> 它为了解决项目启动慢的问题，vite通过一开始将应用中的模块分为依赖和源码两类，改进了开发服务器的启动慢的特点；
>
> 依赖： 大多为在开发时，不会变动的纯js，一些较大的依赖(例如有上百个模块的组件库：element-ui) ,处理的代价很高。依赖通常会存在多种模块化的格式.vite会使用esbuild预构建依赖，esbuld使用Go编写，并且比 js编写的打包器，速度快10-100倍；
>
> 源码： 通常包含一些并非直接是js的文件，需要转换，时常被编译。同时，并不是所有的源码都需要同时被加载。(例如：基于路由拆分的代码模块)。
>
> 以上： 这就是为什么vite启动快的原因；

#### 两者区别

1. vite是基于原生Es6 Modules，在生产环境下打包使用的Rollup；
2. vue-cli基于webpack封装，生产环境和开发环境都是基于webpack打包；
3. 所以两者在生产环境都是基于源代码的文件打包。
4. 在开发环境中，Vite是基于原生的es6，无需对代码进行打包，浏览器可以直接调用，所以说vite因为基于浏览器的原生功能，省掉了打包过程，在开发环境中体验极好；
5. vite会取代vue-cli吗？ 尤雨溪(Evan You)在Twitter上说：
   ​ 起初我不确定，但在这个阶段，我相信最终会是这样。





### 11.版本回退用哪个命令？



### 12.ElementUI和Plus的区别？

// vue版本的区别，ts支持，并举例说明一些组件进行了优化和改写，比如说之前表单组件的布局是用的float布局，后来改写为flex布局，并完善了很多组件

#### 对移动端的支持

> [Element-UI](https://so.csdn.net/so/search?q=Element-UI&spm=1001.2101.3001.7020)对应Element2：基本不支持手机版
>
> [Element-Plus](https://so.csdn.net/so/search?q=Element-Plus&spm=1001.2101.3001.7020)对应Element3：组件布局考虑了手机版展示

#### 框架区别

> Element-ui适用于[Vue2](https://so.csdn.net/so/search?q=Vue2&spm=1001.2101.3001.7020)框架
>
> Element-plus适用于Vue3框架

- 支持TS
- 对部分组件进行优化：之前表单组件的布局是用的float布局，后来改写为flex布局，并完善了很多组件



### 13.对vue.config.js的理解

vue.config.js 是一个可选的配置文件，如果项目的 (和 package.json 同级的) 根目录中存在这个文件，那么它会被 @vue/cli-service 自动加载。你也可以使用 package.json 中的 vue 字段，但是注意这种写法需要你严格遵照 JSON 的格式来写。







### 14.了解nginx吗？

> Nginx是开源、高性能、高可靠的 Web 和反向代理服务器，而且支持热部署，几乎可以做到 7 * 24 小时不间断运行，即使运行几个月也不需要重新启动，还能在不间断服务的情况下对软件版本进行热更新。性能是 Nginx 最重要的考量，其占用内存少、并发能力强、能支持高达 5w 个并发连接数，最重要的是， Nginx 是免费的并可以商业化，配置使用也比较简单。

#### 使用场景

1. 静态资源服务，通过本地文件系统提供服务；
2. 反向代理服务，延伸出包括缓存、负载均衡等；
3. API 服务， OpenResty。

![img](https://pic4.zhimg.com/80/v2-3693069d0955ea98ea22845d8bf270c7_720w.webp)

```shell
# 开机配置
systemctl enable nginx 
# 开机自动启动
systemctl disable nginx 
# 关闭开机自动启动

# 启动Nginx
systemctl start nginx 
# 启动Nginx成功后，可以直接访问主机IP，此时会展示Nginx默认页面

# 停止Nginxs
ystemctl stop nginx

# 重启Nginx
systemctl restart nginx

# 重新加载Nginx
systemctl reload nginx
```

#### **root**

指定静态资源目录位置，它可以写在 http 、 server 、 location 等配置中。

![img](https://pic4.zhimg.com/80/v2-0b739b17868846c9b8692ebec37c69bb_720w.webp)

#### 正向代理

​	正向代理，意思是一个位于客户端和原始服务器（origin server）之间的服务器，为了从原始服务器取得内容，客户端向代理发送一个请求并指定目标（原始服务器），然后代理向原始服务器转交请求并将获得的内容返回给客户端。

​	正向代理是为我们服务的，即为客户端服务的，客户端可以根据正向代理访问到它本身无法访问到的服务器资源。

​	正向代理对我们是透明的，对服务端是非透明的，即服务端并不知道自己收到的是来自代理的访问还是来自真实客户端的访问。

#### 反向代理

​	反向代理（Reverse Proxy）方式是指以代理服务器来接受 Internet 上的连接请求，然后将请求转发给内部网络上的服务器，并将从服务器上得到的结果返回给 Internet 上请求连接的客户端，此时代理服务器对外就表现为一个反向代理服务器。

​	反向代理是为服务端服务的，反向代理可以帮助服务器接收来自客户端的请求，帮助服务器做请求转发，负载均衡等。

​	反向代理对服务端是透明的，对我们是非透明的，即我们并不知道自己访问的是代理服务器，而服务器知道反向代理在为他服务。

- 隐藏真实服务器；
- 负载均衡便于横向扩充后端动态服务；
- 动静分离，提升系统健壮性。

![img](https://pic1.zhimg.com/80/v2-de5f94ec763d4fb6cd56b94c3c4c8db0_720w.webp)

Nginx 实现负载均衡的策略：

- **轮询策略**：默认情况下采用的策略，将所有客户端请求轮询分配给服务端。这种策略是可以正常工作的，但是如果其中某一台服务器压力太大，出现延迟，会影响所有分配在这台服务器下的用户；
- **最小连接数策略**：将请求优先分配给压力较小的服务器，它可以平衡每个队列的长度，并避免向压力大的服务器添加更多的请求；
- **最快响应时间策略**：优先分配给响应时间最短的服务器；
- **客户端 IP绑定策略**：来自同一个 IP 的请求永远只分配一台服务器，有效解决了动态网页存在的 session 共享问题。

https://zhuanlan.zhihu.com/p/498211898?utm_id=0

### 15.组件缓存/页面缓存

https://blog.csdn.net/qq_38987146/article/details/133356576

#### 组件缓存

直接再需要被缓存的组件外加一层keep-alive标签，这个也是特别实用的方法。

被keep-alive包裹的组件，不会销毁，而是将需要缓存的虚拟dom（一个js对象）存在this.cache缓存中，如果渲染的name符合条件，那么虚拟dom就会从缓存中取出，重新渲染

所以被缓存的组件只在初始化的时候执行created,再次切换到那个组件是不会重新created的
v-show有一点相似，只是v-show是通过display来设置显示与隐藏
keep-alive的几个属性

### 16.项目如何部署？



### 17.登录持久化，过期怎么设置？

Vue实现登录数据的持久化可以通过以下几种方式：

- 使用浏览器的本地存储：Vue可以利用浏览器的本地存储机制（如LocalStorage或SessionStorage）将登录信息保存在用户的浏览器中。当用户刷新页面或关闭浏览器后，可以通过读取本地存储中的登录信息来恢复用户的登录状态。

- 使用Cookie：Vue可以将登录信息保存在Cookie中。Cookie是由服务器在浏览器中创建的一个小文件，可以在浏览器和服务器之间传递数据。通过设置Cookie的有效期，可以实现登录数据的持久化。

- 使用插件或第三方库：Vue有一些插件或第三方库可以帮助实现登录数据的持久化，如vuex-persistedstate、vue-cookies等。这些插件或库提供了一些API或封装了一些方法，可以简化持久化登录数据的操作。
  






### 18.24小时内重新登录？设置过期事件？



### 19.cookie和token的区别？



### 20.vue的路由守卫进行权限管理

- 设置基础路由login和home等页面；

- 登录后从后端获取user,token,rights等数据，并将数据同时存储到vuex和sessionStorage中
- 将后端获取的权限数据（作为不同用户显示不同菜单及不同路由的依据）和路由页面进行映射；
- 写公共方法添加动态路由；
- 在路由全局守卫中判断是否已添加过动态路由，如果已添加直接next()未添加调用添加动态路由方法，并且放行（next({ ...to, replace: true });）；
- 添加动态路由位置第二种方法：全局路由中只负责判断是否有动态路由和放行，调用添加动态路由方法，在登录后立即调用，且以防页面手动刷新时进行404页面，需要在App.vue的create()方法中再次调用



### 21.网络请求怎么封装？

- 基础功能

```js
const service = axios.create({
  baseURL: process.env.VUE_APP_BASE_API, // url = base url + request url
  // withCredentials: true, // send cookies when cross-domain requests
  timeout: 10000, // request timeout
  headers: {
    "Content-Type": "application/json;charset=UTF-8",
    "Content-Type": 'application/x-www-form-urlencoded;charset=UTF-8',
    // "Access-Control-Allow-Origin": process.env.VUE_APP_BASE_API,
  }
})
```

添加BaseURL，timeout，header等基本参数；

- 请求拦截器

检查store中是否有token，如果有将token添加到header中；没有的话，不允许发送请求；

添加公共参数：如果路径中有修改、添加的路径，则将用户名和ID添加到data；

判断请求方法：如果是get方法，将用户cookie和信息添加到params；

- 响应拦截器

解析响应数据：响应数据可能会放在内层，对数据解析将其放保存；

判断响应状态：如果返回相应失败，弹出弹框报错；

判断数据类型：如果是数据流，则进行文件下载流程，创建一个blob对象接收数据流，然后创建a节点，添加下载链接；





### 22.输入框怎么设置防抖优化？

防抖主要是用在输入框用输入输入触发一些请求，搜索一些数据，为了减少请求次数优化性能而采取的措施。
解决的问题：高频的事件操作带来了函数的多次调用影响性能。
原理：在一定的时间间隔N秒后才执行该事件，如果在N秒内又触发了该事件则重新计时。（就是你玩游戏技能有CD时间）
应用场景:
（1）输入框连续输入值后，直等到最后一次输入完才进行相关的事件操作
（2）点赞、表单提交、防止重复提交。







# 3.20一面



### 1.介绍自己和项目



### 2.用过vue的那些技术实现了那些功能？



### 3.项目有哪些难点？



### 4.用过Flutter吗？



### 5.网安（我专业）和计算机专业有哪些一样的课程？



### 6.数据库有了解吗？解释一下范式？

> 本科学的早就忘完了，逆天



### 反问

### 1.岗位技术栈

Flutter+Vue ToB多

### 2.可以给个面评吗？

不行

### 3.岗位工作内容？

