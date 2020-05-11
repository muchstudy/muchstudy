---
title: 手写JS函数的call、apply、bind实现
date: 2019-12-01 00:00:02
tags:
---

<div style="width:70%;margin:auto">
{% asset_img title.jpg %}
</div>

&emsp;&emsp;之所以要写这篇，是因为曾经面试被要求在白纸上手写bind实现

&emsp;&emsp;结果跟代码一样清晰明确，一阵懵逼，**没写出来！**

&emsp;&emsp;下面，撸起袖子就是干！~

&emsp;&emsp;把call、apply、bind一条龙都整一遍！~~

### call

#### 定义与使用

> Function.prototype.call(): https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/call

```javascript
// Function.prototype.call()样例
function fun(arg1, arg2) {
  console.log(this.name)
  console.log(arg1 + arg2)
}
const _this = { name: 'YIYING' }
// 接受的是一个参数列表;方法立即执行
fun.call(_this, 1, 2)
```
```shell
// 输出：
YIYING
3
```

#### 手写实现

```javascript
/**
 * 自定义call实现
 * @param context   上下文this对象
 * @param args      动态参数
 */
Function.prototype.ownCall = function(context, ...args) {
  context = (typeof context === 'object' ? context : window)
  // 防止覆盖掉原有属性
  const key = Symbol()
  // 这里的this为需要执行的方法
  context[key] = this
  // 方法执行
  const result = context[key](...args)
  delete context[key]
  return result
}
```

```javascript
// 验证样例
function fun(arg1, arg2) {
  console.log(this.name)
  console.log(arg1 + arg2)
}
const _this = { name: 'YIYING' }
// 接受的是一个参数列表;方法立即执行
fun.ownCall(_this, 1, 2)
```

```shell
// 输出：
YIYING
3
```


### apply

#### 定义与使用

> Function.prototype.apply(): https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/apply

```javascript
// Function.prototype.apply()样例
function fun(arg1, arg2) {
  console.log(this.name)
  console.log(arg1 + arg2)
}
const _this = { name: 'YIYING' }
// 参数为数组;方法立即执行
fun.apply(_this, [1, 2])
```
```shell
// 输出：
YIYING
3
```

#### 手写实现
```javascript
/**
 * 自定义Apply实现
 * @param context   上下文this对象
 * @param args      参数数组
 */
Function.prototype.ownApply = function(context, args) {
  context = (typeof context === 'object' ? context : window)
  // 防止覆盖掉原有属性
  const key = Symbol()
  // 这里的this为需要执行的方法
  context[key] = this
  // 方法执行
  const result = context[key](...args)
  delete context[key]
  return result
}
```

```javascript
// 验证样例
function fun(arg1, arg2) {
  console.log(this.name)
  console.log(arg1 + arg2)
}
const _this = { name: 'YIYING' }
// 参数为数组;方法立即执行
fun.ownApply(_this, [1, 2])
```

```shell
// 输出：
YIYING
3
```

### bind

#### 定义与使用

> Function.prototype.bind()
: https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/bind

```javascript
// Function.prototype.bind()样例
function fun(arg1, arg2) {
  console.log(this.name)
  console.log(arg1 + arg2)
}
const _this = { name: 'YIYING' }
// 只变更fun中的this指向，返回新function对象
const newFun = fun.bind(_this)
newFun(1, 2)
```
```shell
// 输出：
YIYING
3
```

#### 手写实现

```javascript
/**
 * 自定义bind实现
 * @param context     上下文
 * @returns {Function}
 */
Function.prototype.ownBind = function(context) {
  context = (typeof context === 'object' ? context : window)
  return (...args)=>{
    this.call(context, ...args)
  }
}
```

```javascript
// 验证样例
function fun(arg1, arg2) {
  console.log(this.name)
  console.log(arg1 + arg2)
}
const _this = { name: 'YIYING' }
// 只变更fun中的this指向，返回新function对象
const newFun = fun.ownBind(_this)
newFun(1, 2)
```

```shell
// 输出：
YIYING
3
```

> 最后，麻烦以后面试不要再考这道题了！~~~

<div style="width:70%;margin:auto">
<img src='http://muchstudy.com/2020/04/04/%E8%81%8A%E8%81%8A%E4%B8%80%E7%BA%BF%E5%BC%80%E5%8F%91%E7%9A%84%E5%9F%BA%E6%9C%AC%E7%B4%A0%E5%85%BB/%E5%85%AC%E4%BC%97%E5%8F%B7%E4%BA%8C%E7%BB%B4%E7%A0%81.gif'>
</div>
