---
title: Javascript类型与类型判断
date: 2020-07-31 23:32:39
tags:
---

JS类型与类型判断是JS中的基础，有必要归纳总结整理一下。

> 对于这样一篇文章实际上是个人资料库里的一个整理文档，一直在想这么一篇烂大街的文章分享出来是不是有水文的嫌疑？但是，转过头一想，如果大家把文章中的外链都翻过一遍，那就会觉得：**一个不起眼的小点也有它的价值。**


## JS类型
JS共有8种类型，如下表所示

7种[基本类型](https://developer.mozilla.org/zh-CN/docs/Glossary/Primitive)

- Boolean
- Null
- Undefined
- Number
- BigInt
- String
- Symbol

> 基本类型（基本数值、基本数据类型）是一种既非对象也无方法的数据。
> 所有基本类型的值都是不可改变的。但需要注意的是，基本类型本身和一个赋值为基本类型的变量的区别。变量会被赋予一个新值，而原值不能像数组、对象以及函数那样被改变。

剩余一种
- Object

> 除 Object 以外的所有类型都是不可变的（值本身无法被改变）
>
> JavaScript 数据类型和数据结构：[https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Data_structures](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Data_structures)

## 类型判断
### 基本类型判断
- Boolean
```javascript
typeof true // boolean
```

- Number
```javascript
typeof 1 // number
```

- String
```javascript
typeof '1' // string
```

- Undefined
```javascript
typeof undefined // undefined
undefined === undefined // true
```

- Null
```javascript
typeof null // object
null === null // true
```
> 为什么`typeof null`为Object呢？答案如下：
> 在 JavaScript 最初的实现中，JavaScript 中的值是由一个表示类型的标签和实际数据值表示的。对象的类型标签是 0。由于 null 代表的是空指针（大多数平台下值为 0x00），因此，null 的类型标签是 0，typeof null 也因此返回 "object"。（参考来源）    
> 曾有一个 ECMAScript 的修复提案（通过选择性加入的方式），但被拒绝了。该提案会导致 typeof null === 'null'。
>
> Why is typeof null “object”? : [https://stackoverflow.com/questions/18808226/why-is-typeof-null-object](https://stackoverflow.com/questions/18808226/why-is-typeof-null-object)  
> [https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/typeof#null](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/typeof#null)

- BigInt
可以用在一个整数字面量后面加 n 的方式定义一个 BigInt ，如：10n，或者调用函数BigInt()。
```javascript
typeof 10n === 'bigint'; // true
10n == 10 // true
10n === 10 // false
typeof BigInt('1') === 'bigint'; // true
```
> BigInt: [https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/BigInt](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/BigInt)

- Symbol
```javascript
typeof Symbol() // symbol
```

> 由上可以看出，基本类型除了null之外，都可以使用typeof判断出来具体类型.
>
> Why is typeof null “object”? : [https://stackoverflow.com/questions/18808226/why-is-typeof-null-object](https://stackoverflow.com/questions/18808226/why-is-typeof-null-object)

### Object类型判断

- Array
```javascript
Array.isArray([1,2]) // true
```

- Function
```javascript
typeof (()=>{}) // function
```

对于像Date、Math等对象，就没有很好的直接判断方式了，但是可以通过`Object.prototype.toString.call(obj)`来判断

```javascript
Object.prototype.toString.call([]); // [object Array]
Object.prototype.toString.call({}); // [object Object]
Object.prototype.toString.call(''); // [object String]
Object.prototype.toString.call(new Date()); // [object Date]
Object.prototype.toString.call(1); // [object Number]
Object.prototype.toString.call(function () {}); // [object Function]
Object.prototype.toString.call(/test/i); // [object RegExp]
Object.prototype.toString.call(true); // [object Boolean]
Object.prototype.toString.call(null); // [object Null]
Object.prototype.toString.call(); // [object Undefined]
```

> Understanding JavaScript types and reliable type checking.
> [https://ultimatecourses.com/blog/understanding-javascript-types-and-reliable-type-checking](https://ultimatecourses.com/blog/understanding-javascript-types-and-reliable-type-checking)


## 类型判断工具类

通过上面的总结分析，可以考虑封装一个类型判断的工具类，下面是感觉最简洁高效的一个实现：
```javascript
var type = (function(global) {
    var cache = {};
    return function(obj) {
        var key;
        return obj === null ? 'null' // null
            : obj === global ? 'global' // window in browser or global in nodejs
            : (key = typeof obj) !== 'object' ? key // basic: string, boolean, number, undefined, function
            : obj.nodeType ? 'object' // DOM element
            : cache[key = ({}).toString.call(obj)] // cached. date, regexp, error, object, array, math
            || (cache[key] = key.slice(8, -1).toLowerCase()); // get XXXX from [object XXXX], and cache it
    };
}(this));
```
这样使用
```javascript
type(function(){}); // -> "function"
type([1, 2, 3]); // -> "array"
type(new Date()); // -> "date"
type({}); // -> "object"
```

> The most accurate way to check JS object's type? [https://stackoverflow.com/questions/7893776/the-most-accurate-way-to-check-js-objects-type](https://stackoverflow.com/questions/7893776/the-most-accurate-way-to-check-js-objects-type)

## 其它

- NaN
注意NaN不是一种数据类型。  
NaN是一个全局对象的属性，它表示不是一个数字（Not-A-Number）。

NaN有如下特性：
```javascript
typeof NaN // number
NaN === NaN // false
isNaN(NaN) // true,可以通过此种方式来判断NaN
var num = Number('num') // 此时num为NaN
num === num // flase
```

> NaN: [https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/NaN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/NaN)
> Why does typeof NaN return 'number'? :[https://stackoverflow.com/questions/2801601/why-does-typeof-nan-return-number](https://stackoverflow.com/questions/2801601/why-does-typeof-nan-return-number)


<div style="width:70%;margin:auto">
<img src='http://muchstudy.com/2020/04/04/%E8%81%8A%E8%81%8A%E4%B8%80%E7%BA%BF%E5%BC%80%E5%8F%91%E7%9A%84%E5%9F%BA%E6%9C%AC%E7%B4%A0%E5%85%BB/%E5%85%AC%E4%BC%97%E5%8F%B7%E4%BA%8C%E7%BB%B4%E7%A0%81.gif'>
</div>
