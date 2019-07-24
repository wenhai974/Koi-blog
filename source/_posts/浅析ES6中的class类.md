---
title: 浅析ES6中的class类
date: 2019-07-24
categories:
- [JS]
tags:
- [构造函数]
- [ES6]
---

# 浅析ES6中的class类

**JavaScript 语言中，生成实例对象的传统方法是通过构造函数的方式。而ES6 提供了更好的方案，引入了 Class（类）这个概念，作为对象的模板。通过`class`关键字，可以定义类。**

**基本上，ES6 的`class`可以看作只是一个语法糖，它的绝大部分功能，ES5 都可以做到，新的`class`写法只是让对象原型的写法更加清晰、更像面向对象编程的语法而已。**



#### 什么是构造函数

在JS中，构造函数和普通函数在定义上并无区别，是否被称为构造函数在于调用方式的不同，**当使用 new 关键词调用函数时，即称为构造函数**。

在写法上，函数名通常约定以大写字母开头，代码如下：

```javascript
function Person(name, email) {
  this.name = name
  this.email = email
}
Person.prototype.write = function () {
  console.log('我的名字是' + this.name + '，邮箱地址是：' + this.email)
}
var Koi = new Person('一尾小锦鲤', 'koi@163.com')
Koi.constructor === Person // true
Person === Person.prototype.constructor // true
// 这种调用方式，Koi是Person的实例化对象，Person即为Koi的构造函数即构造器。

```

##### 构造函数生成实例的执行过程

```js
1.当使用了构造函数，并且new 构造函数(),后台会隐式执行new Object()创建对象;
2.将构造函数的作用域给新对象，（即new Object()创建出的对象），而函数体内的this就代表new Object()出来的对象。
3.执行构造函数的代码。
4.返回新对象（后台直接返回）;
```

#### class类的写法

同样的，以class的方式改写上面的代码如下：

```javascript
class Person {
  constructor (name, email) {
    this.name = name
    this.email = email
    this.
  }
  write () {
    console.log('我的名字是' + this.name + '，邮箱地址是：' + this.email)
  }
}
var Koi = new Person('一尾小锦鲤', 'koi@163.com') // 使用方法与构造函数的方式完全一致
typeof Person // function
Koi.constructor === Person // true
Person === Person.prototype.constructor // true--函数的原型对象的构造函数即为函数自身

```

上面的代码定义了一个“类”，里面有一个`constructor`的方法，此方法为即为class写法的构造方法，里面的this关键字即代表实例对象，`constructor`为默认方法，当未定义`constructor`方法时，系统会默认添加此方法，当然，此时便无法传入初始化参数。

class类除了构造方法，还自定义了一个`write`方法。注意，定义“类”的方法的时候，前面不需要加上`function`这个关键字，直接把函数定义放进去了就可以了。另外，方法之间不需要逗号分隔，加了会报错。

上面代码表明，类的数据类型就是函数，类本身就指向构造函数。

使用的时候，也是直接对类使用`new`命令，**跟构造函数的用法完全一致**。

在上面代码中，像`write`这样定义的方法，实际上与定义在构造函数的`prototype`上的方法一样。

```javascript
class Person {
  constructor() {
    // ...
  }

  toString() {
    // ...
  }

  toValue() {
    // ...
  }
}

// 等同于

Person.prototype = {
  constructor: Person,
  toString() {},
  toValue() {},
}

```

在类的实例上面调用方法，其实就是调用原型上的方法。

```javascript
class B {}
let b = new B();

b.constructor === B.prototype.constructor // true
```

上面代码中，`b`是`B`类的实例，它的`constructor`方法就是`B`类原型的`constructor`方法。

由于类的方法都定义在`prototype`对象上面，所以类的新方法可以添加在`prototype`对象上面。`Object.assign`方法可以很方便地一次向类添加多个方法。

```javascript
class Person {
  constructor(){
    // ...
  }
}

Object.assign(Person.prototype, {
  toString(){},
  toValue(){}
})
```

另外，因为类的内部所定义的方法都是绑定在`prototype`上，所以早实例对象中都是不可枚举的，**但是在`constructor`函数里面通过`this`关键词绑定的方法，是可枚举的。**这里我们可以理解成在`constructor`中定义的方法以及属性，因为是直接绑定给`this`的，所以在实例化的过程中将直接赋值实例对象。

```javascript
class Point {
  constructor (size, color) {
    this.size = size
    this.color = color
    this.write = function () {
      console.log('我的尺寸是' + this.size + ' 我的颜色是:' + this.color)
    }
  }
  toString () {
    console.log('调用原型上的方法toString')
  }
}
let p = new Point('180', 'red')
p.hasOwnProperty('size') // true
p.hasOwnProperty('write') // true
p.hasOwnProperty('toString') // false
p.write() // 我的尺寸是180 我的颜色是:red
p.toString() // 调用原型上的方法toString
```

这一点在实例化对象中，属性是否可枚举与`构造函数`相同，但是在构造函数和class方法的`prototype`原型对象上，却不同。

```javascript
class Point {
  constructor(x, y) {
    // ...
  }

  toString() {
    // ...
  }
}

Point.prototype.hasOwnProperty('toString') // false
```

使用构造函数时：`toString`方法为可枚举属性

```javascript
var Point = function (x, y) {
  // ...
};

Point.prototype.toString = function() {
  // ...
};
Point.prototype.hasOwnProperty('toString') // true
```

