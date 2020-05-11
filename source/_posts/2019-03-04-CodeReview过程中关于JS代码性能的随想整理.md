---
title: CodeReview过程中关于JS代码性能的随想整理
date: 2019-03-04 22:59:48
tags:
---

## 问题

&emsp;&emsp;团队中做code review有一段时间了，最近一直在思考一个问题，抛开业务逻辑，单纯从代码层面如何评价一段代码的好坏？

&emsp;&emsp;好和坏都是相对的，一段不那么好的代码经过优化之后，如何标准化的给出重构前后的差异呢？

&emsp;&emsp;我们所有的代码都跑在计算机上，计算机的核心是CPU和内存。从这个角度来看，效率高的代码应当占用更少的CPU时间，更少的内存空间。

&emsp;&emsp;因此，问题就演变为优化一段代码，到底优化了多少CPU的使用以及内存空间的使用？

## CPU-时间复杂度
&emsp;&emsp;在数据结构与算法中，常用大O来表示算法的时间复杂度，常见的时间复杂度如下所示：（来源《算法》第四版）

{% asset_img 时间复杂度.jpg 时间复杂度 %}

&emsp;&emsp;时间复杂度这个东西，是描述一个算法在问题规模不断增大时对应的时间增长曲线。所以，这些增长数量级并不是一个准确的性能评价，可以理解为一个近似值，时间的增长近似于logN、NlogN的曲线。如下图所示：

{% asset_img 增长曲线.jpg 增长曲线 %}

&emsp;&emsp;上面是关于时间复杂度的解释，下面通过具体样例来看看代码的时间复杂度

代码一：
```javascript
(function count(arr=[1,2,3,4,5,6,7,8,9,10]){
  let num = 0
  for(let i=0;i<arr.length;i++){
    let item = arr[i]
    num = num + item
  }
  return num
})()
```
&emsp;&emsp;这是一段求数组中数字总和的代码，我们粗略估计上述代码在CPU中表达式运算的时间都是一样的，计为avg_time，那么我们来算一下上面的代码需要多少个avg_time.

&emsp;&emsp;首先从第二行开始，表达式赋值计为1个avg_time；代码的3、4、5行分别要运行10次，其中第三行比较特殊，每次运行需要计算arr.length以及i++,所以这里需要`（2+1+1)*10` 个avg_time;总共就是`（2+1+1)*10+1=41`个avg_time

&emsp;&emsp;接着，我们来对上面的代码优化一番，如下所示：
代码二
```javascript
(function count(arr=[1,2,3,4,5,6,7,8,9,10]){
  let num = 0
  let len = arr.length
  while(len--){
    num = num + arr[len]
  }
  return num
})()
```
&emsp;&emsp;不难算出，优化后的代码只耗费了`1+1+(1+1)*10=22`个avg_time,代码二相对于代码一，节约了`41-22=19`个avg_time,代码性能提升`19/41=46.3%`!

### 如何写出低时间复杂度的代码？

**1.灵活使用break、continue、return**

&emsp;&emsp;这三个关键字一般用在减少循环次数，达到目的，立即退出。如下所示：
```javascript
    (function check(arr=[1,2,3],target=2){
      let len = arr.length
      while(len--){
        if(arr[len]===target){
          // 不再继续后续循环
          return len
        }
      }
      return -1
    })()
```
**2.空间换时间**  

   常见的做法是利用缓存，把上次的计算结果存起来，避免重复计算。


**3.更优的数据结构与算法**

  根据不同的情况选择合适的数据结构与算法，例如，如果需要频繁的从一组数据中通过关键key查询出数据，如果要从json对象和数组中选择，那么可以优先考虑使用json对象来避免数组的遍历查询。

## 内存-空间复杂度

&emsp;&emsp;评价一段代码，除了看它执行需要多少时间，还需要看看需要多少空间，谈到代码的空间占用，必须就得知道JS的内存管理


&emsp;&emsp;JS的内存管理分为三部分：
  - 内存分配。  
  &emsp;&emsp;这里包含包含代码本身以及静态数据与动态数据所需要的内存，其中代码本身与静态数据会分配在stack上，可变的动态数据会分配在heap上

  - 使用分配的内存。  
  - 内存回收。  

这里，放一张JS Runtime的图

{% asset_img runtime.png runtime %}

### 静态内存分配
&emsp;&emsp;是指stack中内存的分配，基础数据类型的数据就放在stack中。另外，stack是有固定大小的，超过stack的长度，就会报错，所以必须得节约着用。

#### 爆栈
```javascript
// 故意来一次爆栈体验
function foo(){
  foo()
}
foo()
// 结果
VM201:1 Uncaught RangeError: Maximum call stack size exceeded
    at foo (<anonymous>:1:13)
    at foo (<anonymous>:2:3)
    at foo (<anonymous>:2:3)
    at foo (<anonymous>:2:3)
    at foo (<anonymous>:2:3)
    at foo (<anonymous>:2:3)
    at foo (<anonymous>:2:3)
    at foo (<anonymous>:2:3)
    at foo (<anonymous>:2:3)
    at foo (<anonymous>:2:3)
```
&emsp;&emsp;我们是怎么达到爆栈目的的呢？因为所有的函数调用，在内存中都存在一个函数调用栈，我们不断无结束条件的递归调用，最终撑破了stack。

如图所示：
{% asset_img stackoverflow.png stackoverflow %}

#### 函数调用栈

可能你会问怎么证明函数调用栈的存在呢？请看如下代码：
```javascript
function second() {
    throw new Error('function call stack');
}
function first() {
    second();
}
function start() {
    first();
}
start();
// 结果如下
VM266:2 Uncaught Error: function call stack
    at second (<anonymous>:2:11)
    at first (<anonymous>:5:5)
    at start (<anonymous>:8:5)
    at <anonymous>:10:1
```
&emsp;&emsp;从上面的运行结果可以看出函数调用栈的顺序，start先入栈，接着first，最后second；打印顺序为首选打印second，最后打印start；满足栈的先进后出的数据结构特性。

#### 内存占用

&emsp;&emsp;了解上面知识点的核心目的还是在于指导我们写出更优的代码，我们知道基本数据类型都放在栈中，对象都放在堆中。另外，通过《JavaScript权威指南》第六版第三章可以知道，js中的数字都是双精度类型，占64位8个字节的空间，字符占16位2个字节的空间。

&emsp;&emsp;有了这个知识，我们就可以估算出我们的代码大致占用了多少内存空间。

&emsp;&emsp;这些毕竟都是理论知识，不禁要怀疑一下，的确是这样的吗？下面我们利用爆栈的原理，通过代码实际瞧瞧

```javascript
let count = 0
try{
  function foo() {
    count++
    foo()
  }
  foo()
}finally{
  console.log(count)
}
// 最终的打印结果为：15662
```

我们知道一个数字占8个字节，栈的大小固定；稍微变更一下代码
```javascript
let count = 0
try{
  function foo() {
    let local = 58 //数字，占8个字节
    count++
    foo()
  }
  foo()
}finally{
  console.log(count)
}
// 最终的打印结果为：13922
```
那么我们可以利用如下方法算一下栈的总大小
```
N = 栈中单个元素的大小
15662 * N = 13922 * (N + 8) // 两次函数调用，栈的总大小相等
(15662 - 13922) * N = 13922 * 8
1740 * N = 111376
N = 111376 / 1740 = 64 bytes
Total stack size = 15662 * 64 = 1002368 = 0.956 MB
```
注：不通环境可能结果不太一样

接下来，我们来确定一下数字类型是否占8个字节空间
```javascript
let count = 0
try{
  function foo() {
    //数字，占8个字节,这里就占16个字节
    let local = 58
    let local2 = 85
    count++
    foo()
  }
  foo()
}finally{
  console.log(count)
}
// 最终的打印结果为：12530
```

计算一下Number的内存占用大小
```
// 总的栈内存空间/栈中元素数量 = 单个栈元素大小
1002368/12530 = 80
// 对比不带任何额外变量的代码，单个栈元素大小是64，这里新增两个16，加起来正好为80
80 = 64+8+8
```

> 经实际验证，在Chrome、Safari、Node环境下，不论变量的值是什么类型，在stack中都占8个字节。**对于字符串貌似跟预期不太一样**，不论多长的字符串实践表明在stack中都占8个字节，**怀疑浏览器默认把字符串转换为了对象，最终占用heap空间**


### 动态内存分配
&emsp;&emsp;是指heap中内存的分配，所有对象都放在heap中，stack中只放对象的引用。

这里有一篇数组占用多少内存空间的文章：[How much memory do JavaScript arrays take up in Chrome?](http://www.mattzeunert.com/2016/07/24/javascript-array-object-sizes.html)

### 如何写出低内存占用的代码？
&emsp;&emsp;低内存占用，从静态内存分配方面可以考虑，尽量少的使用基础类型变量；从动态内存分配的角度，让代码更简洁、不要毫无节制的`new一个对象`、少在对象放东西；

下面是一些小技巧：
**1.三目运算符**
```javascript
    // 条件赋值
    if(a===1){
      b = 'aa'
    }else{
      b = 'bb'
    }
    // 可简化为
    b = a===1 ? 'aa' : 'bb'
```
**2.直接返回结果**
 ```javascript
    if(a===1){
      return true
    }else{
      return false
    }
    // 可简化为
    return a===1
```
一时半会儿想不到好的样例，上面的样例至少节约了代码的空间占用!......欢迎评论补充......

### 内存回收
&emsp;&emsp;我的理解是，当函数调用栈为空时，占用的占内存随之清空；只有堆内存中的数据才需要通过垃圾回收机制来回收。

常见的垃圾回收算法如下：
- 引用计数  
对没有对象的引用计数，如果没有任何外部引用时，则清除该对象；引用计数算法有一个弊端就是**无法清除循坏依赖的对象**。

- 标记清除：  
每次回收，从根对象开始遍历，能遍历到的对象则记为可用，不能遍历到的对象则为需要垃圾回收的对象。此种算法能够解决对象循环依赖的问题。
- 综合算法：  
实际上垃圾回收是一个很复杂的过程，垃圾回收器会根据内存的不通情况采取不同的垃圾回收算法，来实现效率的最大化。

这里有一篇垃圾回收的文章：[A tour of V8: Garbage Collection](http://www.jayconrod.com/posts/55/a-tour-of-v8-garbage-collection) 已经被翻译为了中文，点进去就知道了。

### 如何避免内存溢出？

&emsp;&emsp;从上面的垃圾回收机制不难看出，当某些情况内存无法被回收且不断增加时，内存溢出就会产生。下面是几种常见的会有内存溢出风险的代码。

**1.控制全局变量**  
从垃圾回收的原理我们可以知道，全局变量肯定是不会被回收的。所以我们应当尽量把数据绑定到全局变量上，更应该避免通过用户操作持续的增加全局变量数据的大小。  
另外还需要特别注意意外的全局变量产生，例如：
```javascript
function foo(arg) {
    a = "some text";
    this.b = "some text";
}
// 会在window对象上新增a，b属性
foo()
```

**2.setInterval注意内存占用**  
由于setInterval一直处于活动状态，造成它所依赖的数据一直无法回收。特别容易出现数据越积越多情况

**3.注意闭包**  
闭包里依赖了主函数的数据，为了让闭包续继访问到数据，必须避免当主函数退出时，回收闭包依赖主函数的变量所对应的数据，从而带来内存溢出风险。


>资料：
>1. [JS内存管理](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Memory_Management)
>1. [How JavaScript works: an overview of the engine, the runtime, and the call stack](https://blog.sessionstack.com/how-does-javascript-actually-work-part-1-b0bacc073cf)
>1. [JavaScript stack size](https://glebbahmutov.com/blog/javascript-stack-size/)


<div style="width:70%;margin:auto">
<img src='http://muchstudy.com/2020/04/04/%E8%81%8A%E8%81%8A%E4%B8%80%E7%BA%BF%E5%BC%80%E5%8F%91%E7%9A%84%E5%9F%BA%E6%9C%AC%E7%B4%A0%E5%85%BB/%E5%85%AC%E4%BC%97%E5%8F%B7%E4%BA%8C%E7%BB%B4%E7%A0%81.gif'>
</div>
