---
title: 原型链
date: 2026-03-14 14:54:25
tags:
---

## 前言

 原型链知识点
 <!--more-->
## 正文

### 函数的prototype

>- 函数的prototype属性
> - 每个函数都有一个prototype属性，它默认指向一个Object空对象（即称为：原型对象）
> - 原型对象中有一个属性constructor，它指向函数对象
>- 给原型对象添加属性（一般都是方法）
> - 作用：函数的所有实例对象自动拥有原型对象中的属性（方法）

![](/images/photo1.png)

### 显示原型与隐式原型

>- 每个函数function都有一个prototype，即显示原型
>- 每个实例对象都有一个`__proto__`，可称为隐式原型
>- 对象的隐式原型的值为其对应构造函数的显示原型的值
>- 总结：
> - 函数的prototype属性：在定义函数时自动添加的，默认值是一个空Object对象
> - 对象的`__proto__`：创建对象时自动添加的，默认值为构造函数的prototype属性值
> - 程序员能直接操作显示原型，但不能直接操作隐式原型（ES6之前）（**虽然技术上可以操作隐式原型，但从设计理念和最佳实践来说，我们应该通过构造函数和显式原型来影响原型链，而不是直接操作实例的隐式原型。显示原型：开发者显示操作的属性；隐式原型：自动建立的原型链，通常用于访问属性时沿隐式原型链向上查找**）

![](/images/photo2.png)

### 原型链

>- 原型链
> - 访问一个对象的属性时：
>   - 现在自身属性中查找，找到返回
>   - 如果没有，再沿着`__proto__`这条链向上查找，找到返回
>   - 如果最终没找到，返回undefined（终点Object.prototype）
> - 别名：隐式原型链
> - 作用：查找对象的属性（方法）
>- 构造函数/原型/实体对象的关系（如图）

![](/images/photo3.png)

![](/images/photo4.png)



![](/images/photo5.png)

**补充**：

1. 函数的显示原型指向的对象默认是空Object实例对象（但Object不满足）

   ```js
   console.log(Fun.prototype instanceof Object)//true
   console.log(Object.prototype instanceof Object)//false
   console.log(Function.prototype instanceof Object)//true
   ```

2. 所有函数都是Function的实例（包含Function）

   ```js
   console.log(Function.__proto__===Function.prototype)//true
   ```

3. Object的原型对象是原型链的尽头

   ```js
   console.log(Object.prototype.__proto__)//null
   ```



### 属性问题

>- 读取对象的属性值时，会自动到原型链中查找
>- 设置对象的属性值时，不会查找原型链，如果当前对象上没有此属性，直接添加此属性并设置其值
>- 方法一般定义到原型中，属性一般通过构造函数定义在对象本身上



### 探索instanceof

>- instanceof是如何判断的：
>  - 表达式：A instanceof B
>  - 如果B函数的显示原型对象在A对象的原型链上，则返回true,否则返回false。
>- Function是通过new自己产生的实例

**案例一**：

```js
function Foo() { }
var f1 = new Foo();
console.log(f1 instanceof Foo);//true
console.log(f1 instanceof Object);//true
```

![](/images/photo6.png)

**案例二**：

```js
console.log(Object instanceof Function);//true
console.log(Object instanceof Object);//true
console.log(Function instanceof Function);//true
console.log(Function instanceof Object);//true

function Foo() {}
console.log(Object instanceof Foo);//false
```

![](/images/photo7.png)

### 测试

测试1：

```js
var A = function() {

}
A.prototype.n = 1

var b = new A()

A.prototype = {
n: 2,
m: 3
}

var c = new A()
console.log(b.n, b.m, c.n, c.m)//1,undefined,2,3
```

原理图：

![](./images/photo8.png)

实例对象b沿着隐式原型链向上查找原来的A.prototype指向的原型对象，此时n=1,m=undefined；后面A.prototype被赋值了一个新的对象，也就是说A.prototype指向一个新的对象而`b.__proto__`指向的对象没有发生改变，所以通过此时A构造出来的实例对象的`__proto__`会被A改过后的prototype属性重新赋值，所以c.n=2,c.m=3;（构造函数在创建实例时会将自身的prototype属性赋值给实例对象的`__proto__`属性）

测试2：

```js
var F = function(){};
Object.prototype.a = function(){
console.log('a()')
};
Function.prototype.b = function(){
console.log('b()')
};
var f = new F();
f.a()//a()
f.b()//报错
F.a()//a()
F.b()//b()
```

f是实例对象，f.a()通过隐式查询先找到Object的实例空对象，再找到Object的原型对象里找到方法a,而方法b在Object.prototype里面没有所以报错。而F作为Function的实例对象其`__proto__`指向Function.prototype对象，而Function.prototype对象也是Object的实例对象，所以也可以访问到Object.prototype里面的a方法。