---
title: JavaScript数组倒序算法与性能对比
date: 2017-03-14 11:38:53
categories: 算法
---

&emsp;&emsp;最近正在撸算法第四版，关于JS中数组的倒序，想到如下几种实现方式。

## for push

```javascript
(function (arr) {
    var result = [],len=arr.length-1;
    for(var right=len;right>=0;right--){
        result.push(arr[right])
    }
    console.log(result.join(""));
})("字符倒序测试.".split(""));
```

## for swap half
```javascript
(function (array) {
    var left = null;
    var right = null;
    var length = array.length;
    for (left = 0; left < length / 2; left += 1)
    {
        right = length - 1 - left;
        var temporary = array[left];
        array[left] = array[right];
        array[right] = temporary;
    }
    console.log(array.join(""));
})("字符倒序测试.".split(""));
```

## native reverse
```javascript
(function (arr) {
    var result = arr.reverse();
    console.log(result.join(""));
})("字符倒序测试.".split(""));
```

## 性能对比

&emsp;&emsp;本来想再写一个性能测试的样例，在写之前感觉应该已经有人做过这件事了，所以Google了一下，找到Stackoverflow上<a   href="http://stackoverflow.com/questions/5276953/what-is-the-most-efficient-way-to-reverse-an-array-in-javascript">**这篇答案**</a>，循着找到一个很全面的性能测试样例，点击<a href="https://jsperf.com/js-array-reverse-vs-while-loop/66">**这里**</a>。点击**Run tests**即能在对应的测试环境下测试各种算法的性能情况。

在我本地环境（Testing in Chrome 56.0.2924 / Windows 10 0.0.0）的测试结果如图：

{% asset_img 倒序算法性能对比.png %}
