---
title: 深入理解闭包
categories: 前端
tags:
  - JavaScript
---

### 闭包

#### 什么是闭包

高级程序设计三中:闭包是指有权访问另外一个函数作用域中的变量的函数.可以理解为(**能够读取其他函数内部变量的函数**)

**闭包的作用:** 正常函数执行完毕后,里面声明的变量被垃圾回收处理掉,但是闭包可以让作用域里的 变量,在函数执行完之后依旧保持没有被垃圾回收处理掉

#### 闭包的实例

```javascript
// 创建闭包最常见的方式函数作为返回值
function foo() {
  var name = "kebi";
  return function() {
    console.log(name);
  };
}
var bar = foo();
bar(); //打印kebi    --外部函数访问内部变量
```

**接下来通过一个实例来感受一下闭包的作用:**

接下来实现一个计数器大家肯定会觉得这不是很简单吗

```javascript
var count = 0;

function add() {
  count = count + 1;
  console.log(count);
}
add(); //确实实现了需求
//但是如果需要第二个计数器呢?
//难道要如下这样写吗?
var count1 = 0;

function add1() {
  count1 = count1 + 1;
  console.log(count1);
}
add1(); //确实实现了需求
```

**当我们需要更多地时候,这样明显是不现实的,这里我们就需要用到闭包.**

```javascript
function addCount() {
  var conut = 0;
  return function() {
    count = count + 1;
    console.log(count);
  };
}
```

这里解释一下上边的过程: addCount() 执行的时候, **返回一个函数**, 函数是可以**创建自己的作用域的**, 但是此时返回的这个函数内部需要引用 addCount() **作用域下的变量 count**, **因此这个 count 是不能被销毁的**.接下来需要几个计数器我们就定义几个变量就可以,**并且他们都不会互相影响,每个函数作用域中还会保存 count 变量不被销毁,进行不断的累加**

```javascript
var fun1 = addCount();
fun1(); //1
fun1(); //2
var fun2 = addCount();
fun2(); //1
fun2(); //2
```

#### 闭包常见面试题

```javascript
//闭包for循环
for (var i = 0; i < 4; i++) {
  setTimeout(function() {
    console.log(i);
  }, 300);
}
```

上边打印出来的都是 4, 可能部分人会认为打印的是 0,1,2,3

**原因**:js 执行的时候首先**会先执行主线程,异步相关的会存到异步队列里**,当主线程执行**完毕开始执行异步队列**, 主线程执行完毕后,此时 i 的值为 4,说以在执行异步队列的时候,打印出来的都是 4(这里需要大家对 **event loop 有所了解(js 的事件循环机制)**)

**1.如何修改使其正常打印:(使用闭包使其正常打印)**

```javascript
//方法一:
for (var i = 0; i < 4; i++) {
  setTimeout(
    (function(i) {
      return function() {
        console.log(i);
      };
    })(i),
    300
  );
}
// 或者
for (var i = 0; i < 4; i++) {
  setTimeout(
    (function() {
      var temp = i;
      return function() {
        console.log(temp);
      };
    })(),
    300
  );
}
//这个是通过自执行函数返回一个函数,然后在调用返回的函数去获取自执行函数内部的变量,此为闭包

//方法发二:
for (var i = 0; i < 4; i++) {
  (function(i) {
    setTimeout(function() {
      console.log(i);
    }, 300);
  })(i);
}
// 大部分都认为方法一和方法二都是闭包,我认为方法一是闭包,而方法二是通过创建一个自执行函数,使变量存在这个自执行函数的作用域里
```

**2.真实的获取多个元素并添加点击事件**

```javascript
var op = document.querySelectorAll("p");
for (var j = 0; j < op.length; j++) {
  op[j].onclick = function() {
    alert(j);
  };
}
//alert出来的值是一样的
// 解决办法一:
for (var j = 0; j < op.length; j++) {
  (function(j) {
    op[j].onclick = function() {
      alert(j);
    };
  })(j);
}
// 解决办法二:
for (var j = 0; j < op.length; j++) {
  op[j].onclick = (function(j) {
    return function() {
      alert(j);
    };
  })(j);
}
//解决方法三其实和二类似
for (var j = 0; j < op.length; j++) {
  op[j].onclick = (function() {
    var temp = j;
    return function() {
      alert(j);
    };
  })();
}

//这个例子和例子一几乎是一样的大家可以参考例子一
```

**3.闭包的缺陷:**

通过上边的例子也发现, 闭包会导致内存占用过高,因为变量都没有释放内存