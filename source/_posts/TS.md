---
title: TS
date: 2023-11-12 21:13:45
categories:
  - 前端
tags:
  - 笔记
  - TS
mp3:
cover: img/bg3.jpg
---

# TS

### 基础使用

```shell
tsc --init #初始化配置文件
#vscode终端，运行任务，tsc监视
```

## 一、基础

### 1.基础数据类型

```ts
let num: Number = 10
let num1: number = 1
let u: undefined = undefined
let nu: null = null
let arr: Array<Number> = [] //泛型
let arr1: Number[] = [1, 2, 3]

let obj: Object = {}
obj = null //报错
obj = undefined //报错
obj = 1
obj = []
obj = new String()
obj = String

//服务器返回的数据类型未知
let data: any = 123
data = {}
data = []
data[0].split() //能通过编译，但是不能运行

//void类型
function fun1(): void {
  console.log(123)
}

function getNum(): Number {
  return 1
}
```

类型声明：ts 变量（参数，形参）

### 2.类型推断

> 如果没有明确的指定类型，那么 TypeScript 会推测一个类型

```ts
let t = 123
t = ' ' //报错
```

- 定义变量时，直接给变量赋值，则定义类型为对应的类型
- 定义变量的时候，没有赋值，则定义类型为 any 类型

### 3.联合类型

> 联合类型（Union Types）表示取值可以为多种类型中的一种。

```ts
//flag： 1，true
let flag: boolean | Number = true
flag = 1
```

> 当 TypeScript 不确定一个联合类型的变量到底是哪个类型的时候，我们**只能访问此联合类型的所有类型里共有的属性或方法**

```ts
let flag: boolean | Number | String = true
console.log(flag.split('')) //报错，只会使用共有的属性方法
```

### 4.对象的类型-接口

> TypeScript 中的接口是一个非常灵活的概念，除了可用于[对类的一部分行为进行抽象](https://ts.xcatliu.com/advanced/class-and-interfaces.html#类实现接口)以外，也常用于对「对象的形状（Shape）」进行描述。

```ts
interface Person {
  readonly id: Number
  name: String
  age: Number
  sex?: String
  [propName: string]: String | Number | undefined
}

let tom: Person = {
  id: 1,
  name: 'Tom',
  age: 20,
  gender: 'Female',
}
```

Note:

- 接口首字母大写，接口名前加大写的 I 表示接口
- 定义的属性比接口少是不允许的，多也不可以
- 可选属性含义是该属性可以不存在
- 任意属性一旦定义，**那么确定属性和可选属性的类型都必须是它的类型的子集**：可以使用联合类型；
- 只读属性的约束存在于第一次给对象赋值，而不是第一次给只读属性赋值时；

### 5.数组类型

> `NumberArray` 表示：只要索引的类型是数字时，那么值的类型必须是数字。

```ts
interface INewArray {
  [index: number]: number
}

let newArr: INewArray = [1, 2, 3, 4]
let list: any[] = ['xcatliu', 25, { website: 'http://xcatliu.com' }]
```

### 6.函数类型

> 函数是 JS 中的一等公民；
>
> 输入的参数多或少都不被允许；

```ts
function sum(a: number, b: number): number {
  return a + b
}

interface ISearchFunc {
  //(参数：类型,参数:类型...)：返回值的类型
  (a: string, b: string): boolean
}

const fun: ISearchFunc = function (a: string, b: string): boolean {
  return true
}

let add3: (a: number, b: number) => number = function (a: number, b: number) {
  return a + b
}

let getName = function (name: string, name_second: string): string {
  return name + name_second
}

console.log(getName('Martin', 'Smith'))
console.log(getName('Martin')) //错误
console.log(getName('Martin', 'Smith', 'a')) //错误
```

#### 可选参数和默认参数

> 可选参数必须位于必选参数列表的最后

```ts
let getName = function (name: string, name_second?: string): string {
  return name + name_second
}

console.log(getName('Martin', 'Smith'))
console.log(getName('Martin'))
```

#### 剩余参数

> 剩余参数必须位于参数列表的最后

```ts
let getNameRest = function (
  name: string,
  name_second?: string,
  ...args: number[]
): string {
  return name + name_second
}

console.log(getNameRest('Martin', 'Smith', 123, 45))
```

#### 函数重载

> 函数名相同，形参不同的多个函数

```ts
//使用联合类型实现重载
function myAdd(x: number | string, y: number | string): number | string {
  if (typeof x == 'string' && typeof y == 'string') {
    return x + y //字符串拼接
  } else if (typeof x == 'number' && typeof y == 'number') {
    return x + y //数字相加
  }
}

console.log(myAdd(1, 2))
console.log(myAdd('1', '2'))
```

### 7.断言

> 断言可以用来手动指定一个值的类型

```ts
function getLength(x: string | number): number {
  if ((x as string).length) {
    return (<string>x).length
  } else {
    return x.toString().length
  }
}

console.log(getLength(12345))
console.log(getLength('123'))
```

> 将任何类型断言为 any，非必要不使用

```ts
//给对象添加属性
;(window as any).a = 10
```

> 断言 any 为一个具体类型

```ts
function abc(x: any, y: any): any {
  return x + y
}

let apple = abc(1, 2) as number
let orange = abc('1', '2') as string
```

## 二、进阶

### 1.类型别名

> 为类型名起别名常用于联合类型

```ts
//为类型取别名
type s = string
let str: s = '!123'

//为联合类型命名
type snb = string | number | boolean
let a1: snb = '123'
let a2: snb = true
```

### 2.字符串字面量类型

```ts
//用约束取值只能是某几个字符串中的一个
type stringType = 'Mark' | 'Jacy' | 'Smith'
let myname: stringType = 'Mark'
```

### 3.元组

> 数组合并了相同类型的对象，而元组合并了不同类型的对象

```ts
let Martin: [string, number, boolean] = ['Martin Wang', 24, true]
//添加内容时，需要是指定的类型之一
Martin.push('Yuyu')
Martin.push(undefined)
Martin.push(null)
Martin.push({}) //错误
console.log(Martin)
```

### 4.枚举

> 用于取值被限定的场景
>
> 枚举成员会被赋值为从 `0` 开始递增的数字，同时也会对枚举值到枚举名进行反向映射

```ts
enum NumberType {
  one = 1,
  two = 2,
  three,
  four,
  five,
  six,
  seven,
}
//可以通过名称拿值，也可以通过值拿名称
console.log(NumberType.one)
console.log(NumberType[1])
```

#### 常数项和计算所得项

```ts
enum NumberType {
  one = 1,
  two = 2,
  //计算所得项
  three = 'abc'.length,
  four = 4,
  five,
  six,
  seven,
}
```

#### 常数枚举

> 常数枚举是使用 `const enum` 定义的枚举类型
>
> 常数枚举与普通枚举的区别是，它会在编译阶段被删除，并且不能包含计算成员。

```ts
const enum Directions {
  Up,
  Down,
  Left,
  Right,
}

let directions = [
  Directions.Up,
  Directions.Down,
  Directions.Left,
  Directions.Right,
]
```

#### 外部枚举

> 外部枚举时使用 declare enum 定义的枚举类型

```ts
//在编译结果中会直接删除,主要用于声明文件
declare const enum ABC {
  a,
  b,
  c,
}

let abc_s = ABC.a

console.log(abc_s)
```

### 5.类

> 传统方法里通过构造函数实现类的概念，通过原型链实现继承，ES6 中实现 CLASS
>
> TS 实现 ES6 所有功能，并添加新方法；

```ts
class Animal {
  name: string
  constructor(name: string) {
    this.name = name
  }
  sayHi1() {
    return `My name is ${this.name}`
  }
}

let a11 = new Animal('Jack')
console.log(a11.sayHi1()) // My name is Jack

class Cat extends Animal {
  name: string
  constructor(name: string) {
    super(name)
  }
  sayHi1() {
    return `Miao, ${this.name}`
  }
}

class Dog extends Animal {
  name: string
  constructor(name: string) {
    super(name)
  }
  sayHi1() {
    return `Woof, ${this.name}`
  }
}

let cat = new Cat('Peach')
let dog = new Dog('Puppy')
console.log(cat.sayHi1())
console.log(dog.sayHi1())
```

#### 存取器

> 使用 getter 和 setter 可以控制对象成员的访问

```ts
class Name {
  firstName: string
  lastName: string

  constructor(firstname: string, lastname: string) {
    this.firstName = firstname
    this.lastName = lastname
  }

  //设置存储器
  get fullName() {
    return this.firstName + '-' + this.lastName
  }

  set fullName(value: string) {
    let names = value.split('-')
    this.firstName = names[0]
    this.lastName = names[1]
  }
}

const n = new Name('Martin', 'Smith')
console.log(n.fullName)
n.fullName = 'Martin-Wang'
console.log(n.fullName)
```

#### 静态方法

> 使用 static 修饰符的方法称为静态方法

```ts
class Employee {
  static type: number = 1 //静态成员
  name: string
  job: string

  constructor(name: string, job: string) {
    this.name = name
    this.job = job
  }

  get jobType() {
    if (Employee.type === 1) {
      return 'Employee'
    } else {
      return 'BOSS'
    }
  }
}

class Boss {
  static type: number = 2
  name: string
  static job: string = 'Manger'

  constructor(name: string) {
    this.name = name
  }
  get jobType() {
    if (Employee.type === 1) {
      return 'Employee'
    } else {
      return 'BOSS'
    }
  }
}

console.log(Employee.type)
console.log(Boss.type)
```

#### 其他修饰符`public,private,protected`

> - public 修饰的方法和属性是可以任意访问的；默认值；
> - private 修饰的不能再声明它的类的外部进行访问；
> - protected 修饰的不能外部访问但是在子类可以访问；

```ts
class A {
  public a: string
  private b: string
  protected c: string

  constructor(a: string, b: string, c: string) {
    this.a = a
    this.b = b
    this.c = c
  }

  print() {
    console.log(this.a, this.b, this.c)
  }
}

class B extends A {
  constructor(a: string, b: string, c: string) {
    super(a, b, c)
  }

  print() {
    console.log(this.a)
    console.log(this.b) //错误，b是私有属性
    console.log(this.c)
  }
}
```

> ⚠ 需要注意的是，TypeScript 编译之后的代码中，并没有限制 `private` 属性在外部的可访问性。

#### 类 readonly

> 只允许出现在属性声明或索引签名或构造函数中

```ts
class C {
  readonly d: number //只读属性，但是在构造函数可以修改
  constructor(num: number) {
    this.d = num
  }

  updata() {
    this.d = 10 //报错，只读
  }
}
```

> readonly 定义在参数上，会创建并初始化参数

```ts
  readonly d: number //只读属性，但是在构造函数可以修改
  constructor(readonly d: number) {
    this.d = d
  }
```

#### 抽象类

> abstract 定义抽象类和其中的抽象方法，为子类服务
>
> 提前预定好后代必须具有那些方法和属性；

```ts
abstract class Y {
  name: string
  abstract age: number
  constructor(name: string) {
    this.name = name
    //this.age = 1 不能在构造函数中访问类“Y”中的抽象属性“age”
  }

  abstract sayName(): any //抽象方法在父类不允许具体实现
}

class Z extends Y {
  name: string = 'Martin'
  age: number = 1 //抽象属性在子类必须定义
  constructor(name: string) {
    super(name)
  }

  //抽象方法在子类必须有具体实现
  sayName() {
    console.log(this.name)
  }
}

const z = new Z('z')
console.log(z.name)
z.sayName()
```

#### 类的类型

> 为变量赋予类型

```ts
class Car {
  brand: string
  constructor(brand: string) {
    this.brand = brand
  }
}

class BMW extends Car {
  constructor(name) {
    super(name)
  }
}

const c: Car = new Car('BMW')
const bmw: BMW = new BMW('X1')
```

### 6.类与接口

#### 类实现接口

```ts
interface IResearch {
  research(): string
}

interface IMakeMoney {
  makeMoney(): string
}

interface IEat {
  eat(): string
}

class Professor implements IResearch, IEat, IMakeMoney {
  research(): string {
    return 'Doing a research~'
  }

  makeMoney() {
    return 'Make a lot of money'
  }

  eat(): string {
    return 'Eating'
  }
}

class PostGraduateStudent implements IResearch, IEat {
  research(): string {
    return 'Doing a research~'
  }

  eat(): string {
    return 'Eating'
  }
}

const pro = new Professor()
const stu = new PostGraduateStudent()

console.log(
  pro.research(),
  pro.eat(),
  pro.makeMoney(),

  stu.research(),
  stu.eat()
)
```

#### 接口继承接口

```ts
interface IRun {
  run(): void
}

interface ISwim {
  swim(): void
}

//接口继承可以不实现
interface IActive extends IRun, ISwim {
  jump(): void
}

//类继承接口必须实现
class I implements IActive {
  jump(): void {
    throw new Error('Method not implemented.')
  }
  run(): void {
    throw new Error('Method not implemented.')
  }
  swim(): void {
    throw new Error('Method not implemented.')
  }
}
```

#### 接口继承类

> 接口要继承类中的实例属性和方法
>
> 接口定义的对象必须有对应方法属性

```ts
class NewPerson {
  name: string
  constructor(name: string) {
    this.name = name
  }
  sayHi(): string {
    return this.name
  }
}

interface IPerson extends NewPerson {
  age: number
}

let p1: IPerson = {
  name: 'Martin',
  age: 18,
  sayHi(): string {
    console.log(this.name)
    return this.name
  },
}
```

### 7.声明合并

> 如果定义了两个相同名字的函数、接口或类，那么会合并成一个类型

#### 函数合并

> **//函数合并**
>
> **//重载定义多个函数**

#### 接口合并

> //接口合并
> 接口的属性在合并时会合并到一个接口

```ts
interface Miao {
  name: string
  sex: string
}

interface Miao {
  name: string
  age: number
}

const littleOrange: Miao = {
  name: 'Orange',
  age: 3,
  sex: 'girl',
}
```

#### 类合并

> 与接口合并相同

### 8.泛型

> 指定义函数、接口、类的时候不预先指定具体的类型，在使用的时候在指定类型的一种特性；

> 需求：定一个函数，传入第一个参数是数据，第二个参数是数量

```ts
function getArray<T>(value: T, count: number): T[] {
  const arr: T[] = []
  for (let i = 0; i < count; i++) {
    arr.push(value)
  }
  return arr
}

console.log(getArray(123, 3))
console.log(getArray<string>('123', 3))
console.log(getArray<boolean>(true, 3))

//c++???
function getArray<template>(value: template, count: number): template[] {
  const arr: template[] = []
  for (let i = 0; i < count; i++) {
    arr.push(value)
  }
  return arr
}

console.log(getArray(123, 3))
console.log(getArray<string>('123', 3))
console.log(getArray<boolean>(true, 3))
```

#### 多个类型参数

> 定义泛型时可以一次定义多个类型参数

```ts
function swap<T, U>(tuple: [T, U]): [U, T] {
  return [tuple[1], tuple[0]]
}

swap([7, 'seven'])
```

#### 泛型约束

> 在函数内部使用泛型变量时，由于事先不知道它是那种类型，所以不能随意操作它的属性或方法；

```tsx
function loggingIdentity<T>(arg: T): T {
  console.log(arg.length) //报错
  return arg
}
```

> 对泛型进行约束，只允许这个函数传入包含 length 属性的变量，这就是泛型约束

```ts
interface LengthWise {
  length: number
}

function loggingIdentity<T extends LengthWise>(arg: T): T {
  console.log(arg.length)
  return arg
}
```

> 如果传入的参数不包含 length，编译阶段报错

```ts
interface LengthWise {
  length: number
}

function loggingIdentity<T extends LengthWise>(arg: T): T {
  console.log(arg.length)
  return arg
}

loggingIdentity(4) //报错
loggingIdentity('123') //通过
loggingIdentity(true) //报错
```

#### 泛型接口

```ts
//定义一个泛型接口1
interface IArr {
  <T>(value: T, count: number): Array<T>
}
//泛型参数提前到接口名上
interface IArr<T> {
  (value: T, count: number): Array<T>
}

let getArrs: IArr = function <T>(value: T, count: number): T[] {
  const arr: T[] = []
  for (let i = 0; i < count; i++) {
    arr.push(value)
  }
  return arr
}
```
