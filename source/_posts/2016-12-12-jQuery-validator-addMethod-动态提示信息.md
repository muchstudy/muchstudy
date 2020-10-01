---
title: jQuery validator addMethod 动态提示信息
date: 2016-12-12 15:22:19
categories:
- FrontEnd
---
关于jQuery validator addMethod自定义验证规则网络上大部分都是这样写的
```javascript
$.validator.addMethod('PD_password', function (value, element) {
    var len = value.length;
    if(len<6){
        return false;
    }
    if(len>15){
        return false;
    }
    return true;
}, "密码必须在6-15位之间");
```
&emsp;&emsp;现在，我想要更具体的提示信息，即能准确的提示到底是大于15位还是小于6位。在中文网页搜了一圈都没找到答案，最后在stackoverflow上找到了答案。想了想，感觉有必要把如何实现该需求分享一下。
```javascript
$.validator.addMethod('PD_password', function (value, element) {
    var len = value.length;
    if(len<6){
        $(element).data('error-msg','长度不能少于6位');
        return false;
    }
    if(len>15){
        $(element).data('error-msg','长度不能大于15位');
        return false;
    }
    return true;
}, function(params, element) {
    return $(element).data('error-msg');
});
```
