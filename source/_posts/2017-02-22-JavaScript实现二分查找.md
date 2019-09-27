---
title: JavaScript实现二分查找
date: 2017-02-22 00:59:42
categories: 算法
---

&emsp;&emsp;最近撸《算法》第四版，开篇就是一个Java版本的二分查找算法，下面以JS实现一下。

&emsp;&emsp;二分查找的前提为：数组、有序。逻辑为：优先和数组的中间元素比较，如果等于中间元素，则直接返回。如果不等于则取半继续查找。

```javascript
/**
 * 二分查找，递归实现。
 * @param target
 * @param arr
 * @param start
 * @param end
 * @returns {*}
 */
function binarySearch(target,arr,start,end) {
    var start   = start || 0;
    var end     = end || arr.length-1;

    var mid = parseInt(start+(end-start)/2);
    if(target==arr[mid]){
        return mid;
    }else if(target>arr[mid]){
        return binarySearch(target,arr,mid+1,end);
    }else{
        return binarySearch(target,arr,start,mid-1);
    }
    return -1;
}


/**
 * 有序的二分查找，返回-1或存在的数组下标。不使用递归实现。
 * @param target
 * @param arr
 * @returns {*}
 */
function binarySearch(target,arr) {
    var start   = 0;
    var end     = arr.length-1;

    while (start<=end){
        var mid = parseInt(start+(end-start)/2);
        if(target==arr[mid]){
            return mid;
        }else if(target>arr[mid]){
            start   = mid+1;
        }else{
            end     = mid-1;
        }
    }
    return -1;
}
```


&emsp;&emsp;写完有序，自然而然的想到了无序的情况如何使用二分查找呢？马上想到先使用快排分组，分好组再二分。代码如下：

```javascript
/**
 * 无序的二分查找。返回true/false
 * @param target
 * @param arr
 * @returns {boolean}
 */
function binarySearch(target,arr) {
    while (arr.length>0){
        //使用快速排序。以mid为中心划分大小，左边小，右边大。
        var left    = [];
        var right   = [];
        //选择第一个元素作为基准元素(基准元素可以为任意一个元素)
        var pivot   = arr[0];
        //由于取了第一个元素，所以从第二个元素开始循环
        for(var i=1;i<arr.length;i++){
            var item = arr[i];
            //大于基准的放右边，小于基准的放左边
            item>pivot ? right.push(item) : left.push(item);
        }

        //得到经过排序的新数组
        if(target==pivot){
            return true;
        }else if(target>pivot){
            arr     = right;
        }else{
            arr     = left;
        }
    }
    return false;
}
```

&emsp;&emsp;写完用快速排序实现的无序二分查找，仔细想了一下该算法的时间复杂度，发现还不如直接一个for循环来得快......囧

------
&emsp;&emsp;睡完一觉起来感觉也不是一无是处，这是一个用时间换空间的好办法，大规模问题下有助于节省内存开销。
