---
layout: post
title:  算法笔记
date:   2016-3-4 10:06:37
summary:
categories: algorithms
tags: 算法 笔记
---

解决算法鲁棒性问题最好的方法是在动手写代码之前想好测试用例，考虑到边界条件、特殊输入（比如NULL指针，空字符串等）及错误处理。

<hr>

## 你不知道的Javascript

### 函数作用域和块作用域

表面上看，Javascript并不具有块作用域的相关功能，但是存在以下几种情况：

- with:用with从对象创建的作用域仅在with声明中有效
- try/catch：catch分句会创建一个块级作用域，其中声明的变量仅在catch内部有效
- let: ES6特性，但是使用let进行的声明不会在块级作用域中进行提升（见代码片段）
- const: 可以用来创建块级作用域，其值为固定的常量

{% highlight javascript %}
// let实现块级作用域
{
  let a = 2
  console.log(a)  // 2
}

console.log(a)  // ReferenceError

// ES6之前使用try/catch实现

try {
  throw 2
} catch(a) {
  console.log(a)  // 2
}

console.log(a)  // ReferenceErroe

// let不能使声明提升

{
  console.log(bar)
  let bar = 2
}


// const

if(true) {
  let a = 1
  var b = 2
  const c = 3

  a = 4 // √
  b = 5 // √
  c = 6 // ×
}
{% endhighlight %}

### 提升

问题：

{% highlight javascript %}
// 代码段1
a = 2
var a
console.log(a)  // 输出2

// 代码段2
console.log(a)  // 输出undefined
var a = 2
{% endhighlight %}

解释：

包括变量和函数在内的所有声明都会在任何代码被执行之前首先被处理，例如：

对于var a = 2来说，JS会将其看成两个声明：var a;和a=2;第一个定义声明是在编译阶段进行的，第二个赋值声明会被留在原地等待执行。只有声明本身会被提升，而赋值或其他运行逻辑会留在原地（所以函数声明会被提升，函数表达式不会被提升）。每个作用域都会进行提升操作。

注意：函数声明和变量声明都会被提升，函数会首先被提升，然后才是变量。

### 作用域闭包

闭包是基于词法作用域书写代码时所产生的自然结果。

只要使用了回调函数，实际上就是使用了闭包。函数及作用域

### 关于this

this既不指向函数自身也不指向函数的词法作用域，它在函数被调用时发生绑定，它指向什么完全取决于函数在哪里发生被调用。

### this全面解析

绑定规则

- 默认绑定：直接调用
- 隐式绑定：调用位置有上下文
- 显式绑定：call,apply,bind
- new绑定：new

优先级： new,显示，隐式，默认

例外：

1. 被忽略的this

将null或者unde作为this的绑定对象传入call,apply或者bind，实际运用的是默认绑定规则（即绑定到全局对象，浏览器中为window，会导致一些不和预计的后果，如：修改全局变量）。一种常见的用法是使用apply(...)来展开一个数组，并当作参数传入一个函数，类似的bind可以对参数进行curry化（预先设置一些参数）

针对以上情况的默认绑定，更安全的this使用方法是传递一个空的非委托对象而不是单一的null。

空的委托对象的创建方法：Object.create(null),它和{}很像，但是并不会创建Object.prototype委托，所以比{}更空

2. 间接引用

赋值表达式返回的是目标函数的引用，因此会应用默认绑定，对于默认绑定来说，决定this绑定对象的并不是调用位置是否处于严格模式，而是函数体是否处于严格模式。

{% highlight javascript %}

var Oempty = Object.create(null)

function foo(a,b) {
  console.log('a:' + a + ', b:' + b)
}

//使用apply展开数组
foo.apply(null,[2,3])  // a:2,b:3
foo.apply(Oempty,[2,3])

//ES6中可以使用...来展开数组，eg:
foo(...[2,3])  // a:2,b:3

// bind进行curry化
var bar = foo.bind(null,2);
bar(3)

{% endhighlight %}

软绑定

{% highlight javascript %}

if(!Function.prototype.softBind) {
  Function.prototype.softBind = function(obj) {
  var fn = this
  // 捕获所有curried参数
  var curried = [].slice.call(arguments,1)
  var bound = function() {
  return fn.apply(
    //如果没有this值，或者this值为全局对象，则绑定到当前对象，如果有的话则保持不变
    (!this||this === (window||global))?obj:this,curried.concat.apply(curried,arguments)
  )
  }

  bound.prototype = Object.create(fn.prototype)
  return bound
  }
}

{% endhighlight %}

ES6中的=>（箭头函数）并不是使用function关键字定义的，它不是用this的四种标准规则，而是根据外层（函数或者全局）作用域来决定this。

{% highlight javascript %}
function foo() {
  return (a) => {
  // this继承自foo
  console.log(this.a)
  }
}
{% endhighlight %}


## 对象

在对象中，属性名永远是字符串。

ES6中增加了可计算属性名，可以在文字形式中使用[]包裹一个表达式来当属性名。

{% highlight javascript %}
var prefix = 'foo'

var obj = {
  [prefix + 'bar']:'hello',
  [prefix + 'baz']:'world'
}

obj['foobar'] // hello
obj['bar'] // world
{% endhighlight %}

ES6添加了一中for...of...的遍历数组的方法，可以直接获得属性值，而非for...in...的属性名。数组有内置的@@iterator，也可用内置的@@iterator来遍历数组。

{% highlight javascript %}
var arr = [7,2,3]

for (var i of arr) {
  console.log(i)
}

var it = arr[Symbol.iterator]()

it.next()  // 返回一个对象-> {value:1,done:false}
it.next()
it.next()
it.next()

var it = arr[Symbol.iterator]()

while(!(item=it.next()).done) {
  console.log(item.value)
}
{% endhighlight %}

和数组不同，普通的对象没有内置的@@iterator，所以无法自动完成for...of...的遍历。可以通过自定义的方式来实现：

{% highlight javascript %}
var obj = {
  a:9,
  b:8,
  c:2
}

Object.defineProperty(obj,Symbol.iterator,{
  enumerable:false,
  writable:false,
  configurable:true,
  value:function() {
    var o = this
    var idx = 0
    var ks = Object.keys(o)
    return {
      next: function() {
        return {
          value: o[ks[idx++]],
          done: (idx>ks.length)
      }
    }
  }
  }
})

for (var i of obj) {
  console.log(i)
}

{% endhighlight %}


## 混合对象‘类’

