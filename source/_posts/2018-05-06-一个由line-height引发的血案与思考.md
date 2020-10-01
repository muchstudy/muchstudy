---
title: 一个由line-height引发的血案与思考
date: 2018-05-06 19:32:32
categories:
- 前端
---

## 爆炸

&emsp;&emsp;最近UI走查，发现页面中所有包含文字区块的高度与设计稿中的高度完全不一致，然后UI妹子就爆炸了!

&emsp;&emsp;找了一下原因，发现是由于UI设计稿中设计的文字大部分是`font-size:24px;line-height:24px`,代码实现时为了不至于每处都写一遍字体大小，故直接在根节点上统一设置字体与字体大小为24px，小部分不一致的地方再单独设置字体大小，从而忽略了设置`line-height`为字体的高度。造成的结果就是文字所在的行的行高高于设计稿中的行高。


## 为什么文字行高与字体大小不相等呢？

&emsp;&emsp;翻了一下line-height的<a href='https://www.w3.org/TR/CSS2/visudet.html#propdef-line-height'>官方说明</a>，如下所示：
<div style='text-align:center'>{% asset_img 1.jpg %}<div>

&emsp;&emsp;文档里说line-height的默认值为`normal`,给`normal`的推荐设置值为`1.0到1.2之间`。相当于如果设置了字体的大小而不设置line-height,那么行高默认就为字体的1.0-1.2之间的一个倍数。

&emsp;&emsp;**那么，这个倍数到底具体是多少呢？**

&emsp;&emsp;在chrome的控制台里跟踪了一下，看到项目中引入了`normalize.css`来初始化浏览器的默认css样式。其中，就设置html的`line-height`为`1.15`。

## 为什么normalize.css设置line-height默认为1.15？

&emsp;&emsp;翻了一下github中normalize.css的issues,在593号issues里找到了答案，地址在<a href='https://github.com/necolas/normalize.css/issues/593'>这里</a>。

&emsp;&emsp;大致过程是这样的，有人发现相同字体与大小的文字在不同环境中line-height的值是不一致的，接着，就有人在crossbrowsertesting上做了个测试，得出的结论就是这个问题的的确确存在，而且差异还特别大
> When the font size was 100px, the most common line height was 115px. However, results varied from 101px on Mac Firefox to 136px on Android Chrome.

&emsp;&emsp;最终，由于大部分的行高都为115px,所以，为了解决不同环境中相同字体与字号的文字行高不一致的问题，推荐设置默认`line-height`为1.15

&emsp;&emsp;看到这里，想到一个问题，既然显示设置line-height为1.15是为了解决环境兼容的问题。那么，**为什么不设置`line-height:1`即解决兼容问题，又解决由于行高放大与UI设计稿不符的问题？** 。



## 设置overflow:hidden字体显示不全的问题

&emsp;&emsp;当设置文字`line-height:1`后，再设置文字所在的容器`overflow:hidden`很容易复现文字显示不全的问题，比如下面：

<span style="font-size:100px;line-height:1;overflow:hidden;border:1px solid #ddd;display:inline-block">dphyTxg</span>

代码如下：
```html
<span style="
font-size:100px;
line-height:1;
overflow:hidden;
border:1px solid #ddd;
display:inline-block">dphyTxg</span>
```
注：这里可以使用div演示，然后去掉display:inline-block。使用div发现不知道为毛markdown里的样例乱了...

## 为什么设置行高与字体大小一致文字会显示不全？

&emsp;&emsp;到这里，才真正进入到深层次的原因探究。

&emsp;&emsp;对于这个问题，需要先了解字体度量，引用一下其它的文章说明，如下所示：

<div style='text-align:center'>{% asset_img 2.jpg %}<div>

1. <a href='https://zhuanlan.zhihu.com/p/25808995'>原文地址</a>
1. <a href='http://iamvdo.me/en/blog/css-font-metrics-line-height-and-vertical-align'>Deep dive CSS: font metrics, line-height and vertical-align</a>
1. <a href='http://designwithfontforge.com/zh-CN/The_EM_Square.html'>EM Square</a>
1. <a href='https://fontforge.github.io/en-US/'>FontForge</a>


简要解释为如下几点：
1. 我们设置`font-size`的高度实际上是对应字体的`EM square`部分
1. 字体在设计时可以超出`EM square`部分
1. 字体实际设计与`EM square`部分一致时，`line-height:normal`与`line-height:1`相等
1. 从上图中可以看到，`ascender+descender>Em Size` 即`1100+540>1000`,此时`line-height:1.64`,所以100px的文字默认的行高为164px

## 样例分析

&emsp;&emsp;下面就以window环境下的默认字体微软雅黑为例实际看看line-height的计算。

<h1 id="font" style="line-height:normal;font-size: 100px;font-family: 微软雅黑">h1</h1>
<p>line-height: <code id="o1" style="font-size: 50px"></code></p>
<script>
  document.getElementById('o1').textContent = window.getComputedStyle(document.getElementById('font')).height;
</script>

代码如下：
```html
<h1 id="font" style="line-height:normal;font-size: 100px;font-family: 微软雅黑">h1</h1>
<p>line-height: <code id="o1" style="font-size: 50px"></code></p>
<script>
  document.getElementById('o1').textContent = window.getComputedStyle(document.getElementById('font')).height;
</script>
```
&emsp;&emsp;从上面可以看到微软雅黑100px情况在，在windows下行高显示为132px。通过FontForge来核对一下看看是不是这样的。

<div style='text-align:center'>{% asset_img 3.jpg %}<div>

&emsp;&emsp;从上图中可以看到，使用`（ascender+descender)/Em Size`,即`(2167+536)/2048≈1.32`，也就是line-height为1.32.


## 结论

1. `line-height:normal`的值跟字体有关系，相同字体在不同环境也不一样
1. 当line-height设置的值小于默认的值时就会存在显示不全的问题
1. normalize.css设置line-height默认为1.15，当字体line-height的normal大于1.15就会存在文字显示不全的问题
1. 要解决字体在不同平台line-height不一致的问题，需根据具体字体，选择normal在不同平台上的最大值设置


<div style="width:70%;margin:auto">
<img src='http://muchstudy.com/2020/04/04/聊聊一线开发的基本素养/公众号二维码.gif'>
</div>
