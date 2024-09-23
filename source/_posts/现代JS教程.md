# 现代JS教程

## 一、JS编程语言

### 1.简介

现代的 JavaScript 是一种“安全的”编程语言。它不提供对内存或 CPU 的底层访问，因为它最初是为浏览器创建的，不需要这些功能。

JavaScript 的能力很大程度上取决于它运行的环境。例如，[Node.js](https://wikipedia.org/wiki/Node.js) 支持允许 JavaScript 读取/写入任意文件，执行网络请求等的函数。

- 与 HTML/CSS 完全集成。
- 简单的事，简单地完成。
- 被所有的主流浏览器支持，并且默认开启。

### 2.手册与规范



### 3.“use strict”

ES5 规范增加了新的语言特性并且修改了一些已经存在的特性。为了保证旧的功能能够使用，大部分的修改是默认不生效的。你需要一个特殊的指令 —— `"use strict"` 来明确地激活这些特性。

请确保 `"use strict"` 出现在脚本的最顶部，否则严格模式可能无法启用。



### 4.变量

Note:**声明两次会触发 error**

一个变量应该只被声明一次。

对同一个变量进行重复声明会触发 error：

#### 变量命名

JavaScript 的变量命名有两个限制：

1. 变量名称必须仅包含字母、数字、符号 `$` 和 `_`。
2. 首字符必须非数字。

如果命名包括多个单词，通常采用驼峰式命名法（[camelCase](https://en.wikipedia.org/wiki/CamelCase)）。也就是，单词一个接一个，除了第一个单词，其他的每个单词都以大写字母开头：`myVeryLongName`。

Note：允许非英文的命名

Note：保留字

```js
let let = 5; // 不能用 "let" 来命名一个变量，错误！
let return = 5; // 同样，不能使用 "return"，错误！
```

```js
// 注意：这个例子中没有 "use strict"

num = 5; // 如果变量 "num" 不存在，就会被创建

alert(num); // 5
```

```js
"use strict";

num = 5; // 错误：num 未定义
```

#### 常量

```js
const COLOR_RED = "#F00";
```

作为一个“常数”，意味着值永远不变。但是有些常量在执行之前就已知了（比如红色的十六进制值），还有些在执行期间被“计算”出来，但初始赋值之后就不会改变。

### 5.数据类型

在 JavaScript 中有 8 种基本的数据类型（译注：7 种原始类型和 1 种引用类型）

#### Number

```js
alert( 1 / 0 ); // Infinity
alert( "not a number" / 2 ); // NaN，这样的除法是错误的

alert( NaN + 1 ); // NaN
alert( 3 * NaN ); // NaN
alert( "not a number" / 2 - 1 ); // NaN
```

> **数学运算是安全的**
>
> 在 JavaScript 中做数学运算是安全的。我们可以做任何事：除以 0，将非数字字符串视为数字，等等。
>
> 脚本永远不会因为一个致命的错误（“死亡”）而停止。最坏的情况下，我们会得到 `NaN` 的结果。

#### Bigint

在 JavaScript 中，“number” 类型无法安全地表示大于 `(2^53-1)`（即 `9007199254740991`），或小于 `-(2^53-1)` 的整数。

`BigInt` 类型是最近被添加到 JavaScript 语言中的，用于表示任意长度的整数。

```js
// 尾部的 "n" 表示这是一个 BigInt 类型` 
`const` bigInt `=` `1234567890123456789012345678901234567890n``;
```

#### String

在 JavaScript 中，有三种包含字符串的方式。

1. 双引号：`"Hello"`.
2. 单引号：`'Hello'`.
3. 反引号：``Hello``.

#### Boolean

```js
let nameFieldChecked = true; // yes, name field is checked
let ageFieldChecked = false; // no, age field is not checked
```

#### Null

相比较于其他编程语言，JavaScript 中的 `null` 不是一个“对不存在的 `object` 的引用”或者 “null 指针”。



#### Undefined

如果一个变量已被声明，但未被赋值，那么它的值就是 `undefined`

```js
let age;

alert(age); // 弹出 "undefined"
```



#### Object和Symbol

`object` 类型是一个特殊的类型。

其他所有的数据类型都被称为“原始类型”，因为它们的值只包含一个单独的内容（字符串、数字或者其他）。相反，`object` 则用于储存数据集合和更复杂的实体。

`symbol` 类型用于创建对象的唯一标识符。我们在这里提到 `symbol` 类型是为了完整性，但我们要在学完 `object` 类型后再学习它。

#### Typeof运算符

```js
typeof undefined // "undefined"

typeof 0 // "number"

typeof 10n // "bigint"

typeof true // "boolean"

typeof "foo" // "string"

typeof Symbol("id") // "symbol"

typeof Math // "object"  (1)

typeof null // "object"  (2)

typeof alert // "function"  (3)
```

### 6.类型转换



### 7.函数

函数对外部变量拥有全部的访问权限。函数也可以修改外部变量。

```js
let userName = 'John';

function showMessage() {
  userName = "Bob"; // (1) 改变外部变量

  let message = 'Hello, ' + userName;
  alert(message);
}

alert( userName ); // John 在函数调用之前

showMessage();

alert( userName ); // Bob，值被函数修改了
```

只有在没有局部变量的情况下才会使用外部变量。

如果在函数内部声明了同名变量，那么函数会 **遮蔽** 外部变量。



#### 参数

我们可以通过参数将任意数据传递给函数。

当函数在 `(*)` 和 `(**)` 行中被调用时，给定值被复制到了局部变量 `from` 和 `text`。然后函数使用它们进行计算。

换一种方式，我们把这些术语搞清楚：

- 参数（parameter）是函数声明中括号内列出的变量（它是函数声明时的术语）。
- 参数（argument）是调用函数时传递给函数的值（它是函数调用时的术语）。

#### 返回值

**空值的** `return` **或没有** `return` **的函数返回值为** `undefined`

```js
function doNothing() { /* 没有代码 */ }

alert( doNothing() === undefined ); // true
```



### 8.函数表达式

#### 回调函数



### 9.箭头函数





















## 二、浏览器：文档、事件、接口



## 三、其他文章