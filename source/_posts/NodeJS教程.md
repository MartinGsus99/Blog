# NodeJS教程

Node.js 应用在单个进程中运行，不会为每个请求创建新线程。Node.js 在其标准库中提供了一组异步 I/O 原语，可防止 JavaScript 代码阻塞，并且通常，Node.js 中的库是使用非阻塞范例编写的，这使得阻塞行为成为例外而不是常态。

### NodeJS与浏览器之间的差异

- 在浏览器中，你所做的大部分时间都是与 DOM 或其他 Web 平台 API（如 Cookies）进行交互。当然，Node.js 中不存在这些。你没有浏览器提供的 `document`、`window` 和所有其他对象。在浏览器中，我们没有 Node.js 通过其模块提供的所有好 API，例如文件系统访问功能。
- 在 Node.js 中你可以控制环境。除非你正在构建任何人都可以部署到任何地方的开源应用，否则你知道将在哪个版本的 Node.js 上运行该应用。与浏览器环境相比，你无法选择访问者将使用哪种浏览器，这非常方便。
- Node.js 同时支持 CommonJS 和 ES 模块系统（自 Node.js v12 以来），而在浏览器中，我们开始看到 ES 模块标准正在实现。

### WebAssembly

[WebAssembly](https://webassembly.org/) 是一种高性能的汇编类语言，可以从各种语言编译，包括 C/C++、Rust 和 AssemblyScript。目前，Chrome、Firefox、Safari、Edge 和 Node.js 均支持它！

WebAssembly 规范详细说明了两种文件格式，一种二进制格式称为 WebAssembly 模块，带有 `.wasm` 扩展名，相应的文本表示称为 WebAssembly 文本格式，带有 `.wat` 扩展名。

#### 关键概念

- 模块：编译后的 WebAssembly 二进制文件，即 `.wasm` 文件。
- 内存：可调整大小的ArrayBuffer
- 表：可调整大小的类型化引用数组未存储在内存中。
- 实例：模块及其内存、表和变量的实例化。

为了使用 WebAssembly，你需要一个 `.wasm` 二进制文件和一组 API 来与 WebAssembly 通信。Node.js 通过全局 `WebAssembly` 对象提供必要的 API。

#### 生成WebAssebly模块

- 手动编写WebAssembly（.wat）并使用wabt工具转换为二进制格式；
- 将emscripten与C/C++应用一起使用；

> 工具不仅生产二进制文件，还生成JS Glue代码和相应的HTML文件；

#### 如何使用

```js
// Assume add.wasm file exists that contains a single function adding 2 provided arguments
const fs = require('node:fs');

const wasmBuffer = fs.readFileSync('/path/to/add.wasm');
WebAssembly.instantiate(wasmBuffer).then(wasmModule => {
  // Exported function live under instance.exports
  const { add } = wasmModule.instance.exports;
  const sum = add(5, 6);
  console.log(sum); // Outputs: 11
});

```

#### 与操作系统交互

> WebAssemmbly模块本身无法直接访问操作系统功能，可以使用第三方工具Wasmtime访问；
>
> https://wasi.dev/

## 异步流控制

为什么会发生这种情况？`setTimeout` 指示 CPU 将指令存储在总线上的其他位置，并指示数据计划在稍后时间拾取。在函数在 0 毫秒标记处再次命中之前，经过数千个 CPU 周期，CPU 从总线获取指令并执行它们。唯一的问题是歌曲 ('') 在数千个周期之前被返回。

在处理文件系统和网络请求时也会出现同样的情况。主线程根本无法在一段不确定的时间内被阻止 - 因此，我们使用回调以受控的方式及时安排代码的执行。

#### 系列

> 函数按严格的顺序执行，与for循环相似；

```js
// operations defined elsewhere and ready to execute
const operations = [
  { func: function1, args: args1 },
  { func: function2, args: args2 },
  { func: function3, args: args3 },
];

function executeFunctionWithArgs(operation, callback) {
  // executes function
  const { args, func } = operation;
  func(args, callback);
}

function serialProcedure(operation) {
  if (!operation) process.exit(0); // finished
  executeFunctionWithArgs(operation, function (result) {
    // continue AFTER callback
    serialProcedure(operations.shift());
  });
}

serialProcedure(operations.shift());

```

#### 完全并行

> 当排序不是问题时，例如向1000人发送电子邮件；

```js
let count = 0;
let success = 0;
const failed = [];
const recipients = [
  { name: 'Bart', email: 'bart@tld' },
  { name: 'Marge', email: 'marge@tld' },
  { name: 'Homer', email: 'homer@tld' },
  { name: 'Lisa', email: 'lisa@tld' },
  { name: 'Maggie', email: 'maggie@tld' },
];

function dispatch(recipient, callback) {
  // `sendEmail` is a hypothetical SMTP client
  sendMail(
    {
      subject: 'Dinner tonight',
      message: 'We have lots of cabbage on the plate. You coming?',
      smtp: recipient.email,
    },
    callback
  );
}

function final(result) {
  console.log(`Result: ${result.count} attempts \
      & ${result.success} succeeded emails`);
  if (result.failed.length)
    console.log(`Failed to send to: \
        \n${result.failed.join('\n')}\n`);
}

recipients.forEach(function (recipient) {
  dispatch(recipient, function (err) {
    if (!err) {
      success += 1;
    } else {
      failed.push(recipient.name);
    }
    count += 1;

    if (count === recipients.length) {
      final({
        count,
        success,
        failed,
      });
    }
  });
});

```

#### 有限并行

> 限制总任务中的任务数量

```js
let successCount = 0;

function final() {
  console.log(`dispatched ${successCount} emails`);
  console.log('finished');
}

function dispatch(recipient, callback) {
  // `sendEmail` is a hypothetical SMTP client
  sendMail(
    {
      subject: 'Dinner tonight',
      message: 'We have lots of cabbage on the plate. You coming?',
      smtp: recipient.email,
    },
    callback
  );
}

function sendOneMillionEmailsOnly() {
  getListOfTenMillionGreatEmails(function (err, bigList) {
    if (err) throw err;

    function serial(recipient) {
      if (!recipient || successCount >= 1000000) return final();
      dispatch(recipient, function (_err) {
        if (!_err) successCount += 1;
        serial(bigList.pop());
      });
    }

    serial(bigList.pop());
  });
}

sendOneMillionEmailsOnly();

```



## 阻塞与非阻塞概述

> IO主要指与libuv支持的系统磁盘和网络的交互；

### 阻塞

NodeJs进程中其他 JavaScript 的执行必须等到非 JavaScript 操作完成。

在 Node.js 中，由于占用大量 CPU 而不是等待非 JavaScript 操作（例如 I/O）而表现出较差性能的 JavaScript 通常不被称为阻塞。Node.js 标准库中使用 libuv 的同步方法是最常用的阻塞操作。原生模块也可能有阻塞方法。

Node.js 标准库中的所有 I/O 方法都提供异步版本，它们是非阻塞的，并接受回调函数。某些方法也有阻塞对应项，其名称以 `Sync` 结尾。

#### 比较代码

> 阻塞方法同步执行，非阻塞方法异步执行

```js
  let data = fs.readFileSync('./data.md', 'utf-8')
  
  let data=fs.readFile('/file.md', (err, data) => {
  		if (err) throw err;
  		console.log(data);
	});
```

#### 并发性和吞吐量

> Node.js 中的 JavaScript 执行是单线程的，因此并发是指事件循环在完成其他工作后执行 JavaScript 回调函数的能力。任何预期以并发方式运行的代码都必须允许事件循环在发生非 JavaScript 操作（如 I/O）时继续运行。
>
> 例如，让我们考虑这样一种情况，其中对 Web 服务器的每个请求需要 50 毫秒才能完成，其中 45 毫秒是可以异步完成的数据库 I/O。选择非阻塞异步操作可释放每个请求的 45ms 来处理其他请求。仅通过选择使用非阻塞方法而不是阻塞方法，容量就会有显著差异。

#### 混合阻塞和非阻塞代码的危险

```js
const fs = require('node:fs');

fs.readFile('/file.md', (err, data) => {
  if (err) throw err;
  console.log(data);
});
fs.unlinkSync('/file.md');

```

上述代码可能会在readFile之前删除文件；更好的写法是完全非阻塞，保证正确顺序执行；

## JS异步编程和回调

avaScript 默认是同步的并且是单线程的。这意味着代码无法创建新线程并并行运行。

但 JavaScript 诞生于浏览器内部，它的主要工作是响应用户操作，如 `onClick`、`onMouseOver`、`onChange`、`onSubmit` 等等。如何使用同步编程模型做到这一点？

> 答案在其环境中。浏览器提供了一种方法，通过提供一组可以处理此类功能的 API。



#### 处理回调中的错误？

一种非常常见的策略是使用 Node.js 采用的策略：任何回调函数中的第一个参数都是错误对象：错误优先回调

### NodeJS事件循环



### NodeJS时间触发器

在后端，Node.js 为我们提供了使用 [`events` 模块](https://nodejs.cn/api/events.html) 构建类似系统的选项。

这个模块特别提供了 `EventEmitter` 类，我们将使用它来处理事件。

```js
const EventEmitter = require('node:events');

const eventEmitter = new EventEmitter();

```

- emit触发事件
- on添加在触发事件时要执行的回调函数；
- once一次性监听器
- removeListener()/off()删除事件监听器；
- removeAllListeners()删除事件所有监听器；

### process.nextTick

> 每次事件循环完成一次完整的行程时，我们都将其称为一个勾号。
>
> 当我们将一个函数传递给 `process.nextTick()` 时，我们会指示引擎在当前操作结束时（下一个事件循环开始之前）调用此函数
>
> 事件循环正忙于处理当前函数代码。当此操作结束时，JS 引擎将运行在该操作期间传递给 `nextTick` 调用的所有函数。
>
> 这是我们可以告诉 JS 引擎异步处理函数（在当前函数之后）的方式，但要尽快处理，而不是将其排队。

### setImmediate

> 作为 setImmediate() 参数传递的任何函数都是在事件循环的下一次迭代中执行的回调。

`setImmediate()` 与 `setTimeout(() => {}, 0)`（传递 0ms 超时）以及与 `process.nextTick()` 和 `Promise.then()` 有何不同？

传递给 `process.nextTick()` 的函数将在当前操作结束后，在事件循环的当前迭代中执行。这意味着它将始终在 `setTimeout` 和 `setImmediate` 之前执行。

具有 0ms 延迟的 `setTimeout()` 回调与 `setImmediate()` 非常相似。执行顺序将取决于各种因素，但它们都将在事件循环的下一次迭代中运行。

`process.nextTick` 回调已添加到 `process.nextTick queue`。`Promise.then()` 回调已添加到 `promises microtask queue`。`macrotask queue` 中添加了 `setTimeout`、`setImmediate` 回调。

事件循环首先执行 `process.nextTick queue` 中的任务，然后执行 `promises microtask queue`，然后执行 `macrotask queue`。



## 操作文件

### 统计文件信息

```js
const fs = require('node:fs');

fs.stat('/Users/joe/test.txt', (err, stats) => {
  if (err) {
    console.error(err);
    return;
  }

  stats.isFile(); // true
  stats.isDirectory(); // false
  stats.isSymbolicLink(); // false
  stats.size; // 1024000 //= 1MB
});

```

### 文件路径

```js
const path = require('node:path');

const notes = '/users/joe/notes.txt';

path.dirname(notes); // /users/joe
path.basename(notes); // notes.txt
path.extname(notes); // .txt

```









## 命令行

### NodeJS接受命令行输入

> readline没模块读取process.stdin流；

```js
const readline = require('node:readline')

const rl = readline.createInterface({
    input: process.stdin,
    output: process.stdout
})

rl.question('What do you think of Node.js? ', (answer) => {
    console.log(`Thank you for your valuable feedback: ${answer}`)
    rl.close()
})
```

```shell
npm install inquirer
```

```js
const inquirer = require('inquirer');

const questions = [
  {
    type: 'input',
    name: 'name',
    message: "What's your name?",
  },
];

inquirer.prompt(questions).then(answers => {
  console.log(`Hi ${answers.name}!`);
});

```



### 如何读取环境变量

> process.env属性，托管在启动进程时设置的所有环境变量；



# Express

## 基本路由

```js
app.METHOD(PATH, HANDLER)
```

- METHOD为小写http请求方法；
- PATH时服务器路径；
- HANDLER：路由匹配时执行的函数；

```js
app.get('/', (req, res) => {
    res.send('hello world')
})

app.get('/test', (req, res) => {
    res.send('hello test')
})

app.get('/user', (req, res) => {
    res.send('hello user')
})
```

#### 路由参数

```js
app.get('/user/:uid/books/:boodId', (req, res) => {
    res.send(req.params)
})
```

#### 路由处理程序

> 提供多个回调，行为类似于中间件来处理请求，唯一的例外是这些回调可能会调用next（’route‘）来绕过剩余的路由回调；

```js
app.get('/example/b', (req, res, next) => {
  console.log('the response will be sent by the next function ...')
  next()
}, (req, res) => {
  res.send('Hello from B!')
})
```

#### 响应方法

| 方法                                                         | 描述                                                 |
| ------------------------------------------------------------ | ---------------------------------------------------- |
| [res.download()](https://express.nodejs.cn/en/4x/api.html#res.download) | 提示要下载的文件。                                   |
| [res.end()](https://express.nodejs.cn/en/4x/api.html#res.end) | 结束响应过程。                                       |
| [res.json()](https://express.nodejs.cn/en/4x/api.html#res.json) | 发送 JSON 响应。                                     |
| [res.jsonp()](https://express.nodejs.cn/en/4x/api.html#res.jsonp) | 发送带有 JSONP 支持的 JSON 响应。                    |
| [res.redirect()](https://express.nodejs.cn/en/4x/api.html#res.redirect) | 重定向请求。                                         |
| [res.render()](https://express.nodejs.cn/en/4x/api.html#res.render) | 渲染视图模板。                                       |
| [res.send()](https://express.nodejs.cn/en/4x/api.html#res.send) | 发送各种类型的响应。                                 |
| [res.sendFile()](https://express.nodejs.cn/en/4x/api.html#res.sendFile) | 将文件作为八位字节流发送。                           |
| [res.sendStatus()](https://express.nodejs.cn/en/4x/api.html#res.sendStatus) | 设置响应状态码并将其字符串表示形式作为响应正文发送。 |

#### app.route

> 可以创建链式的路由处理程序；

```js
app.route('/book')
  .get((req, res) => {
    res.send('Get a random book')
  })
  .post((req, res) => {
    res.send('Add a book')
  })
  .put((req, res) => {
    res.send('Update the book')
  })
```

#### express.router

```js
const express = require('express')
const router = express.Router()
const timelog = (req, res, next) => {
    console.log('Time: ', Date.now())
    next()
}

router.use(timelog)

router.get('/', (req, res) => {
    res.send('hello Martin')
})

router.get('/:uid', (req, res) => {
    res.send(`hello ,${req.params.uid}`)
})

router.get('/:uid/books/:boodId', (req, res) => {
    res.send(req.params)
})


module.exports = router
```



## 中间件

> 中间件函数是可以访问应用请求-响应周期中的 [请求对象](https://express.nodejs.cn/en/4x/api.html#req)（`req`）、[响应对象](https://express.nodejs.cn/en/4x/api.html#res)（`res`）和 `next` 函数的函数。`next` 函数是 Express 路由中的一个函数，当被调用时，它会在当前中间件之后执行中间件。
>
> 中间件函数可以执行以下任务：
>
> - 执行任何代码。
> - 更改请求和响应对象。
> - 结束请求-响应周期。
> - 调用堆栈中的下一个中间件。

```js
const timelog = (req, res, next) => {
    console.log('Time: ', Date.now())
    next()
}

router.use(timelog)
```

#### cookie-parser

```js
const express = require('express')
const cookieParser = require('cookie-parser')
const cookieValidator = require('./cookieValidator')

const app = express()

async function validateCookies (req, res, next) {
  await cookieValidator(req.cookies)
  next()
}

app.use(cookieParser())

app.use(validateCookies)

// error handler
app.use((err, req, res, next) => {
  res.status(400).send(err.message)
})

app.listen(3000)
```



### 使用中间件

> Express 是一个路由和中间件 Web 框架，其自身功能最少：Express 应用本质上是一系列中间件函数调用。

中间件可执行任务：

- 执行任何代码；
- 更改请求和响应对象；
- 结束请求和响应周期；
- 调用下一个中间件函数；

分类：

- 应用级中间件：使用 `app.use()` 和 `app.METHOD()` 函数将应用级中间件绑定到 [应用对象](https://express.nodejs.cn/en/4x/api.html#app) 的实例；
- 路由级：路由级中间件的工作方式与应用级中间件相同，只是它绑定到 `express.Router()` 的实例。
- 错误处理中间件：以与其他中间件函数相同的方式定义错误处理中间件函数，但使用四个参数而不是三个参数，特别是使用签名 `(err, req, res, next)`：

```js
app.use((err, req, res, next) => {
  console.error(err.stack)
  res.status(500).send('Something broke!')
})
```

- 内置中间件；

> - express.static 提供静态资源；
> - express.json 使用json有效负载解析传入请求；
> - express.urlencoded 使用url编码的负载解析传入的请求；

- 第三方；

## 防盗链

> 防止外部网站盗用本地网站资源；
>
> 禁止该域名之外的其他网站访问资源；

```js
app.use((req, res, next) => {
    let refer = req.get('referer')
    if (refer) {
        let url = new URL(refer)
        let hostname = url.hostname
        if (hostname === 'localhost' || hostname === '127.0.0.1') {
            next()
        } else {
            res.status(404).send('非法请求')
            return
        }
    }
})
```







