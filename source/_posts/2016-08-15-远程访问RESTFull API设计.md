---
title: 远程访问API设计
date: 2016-08-15 21:59:06
categories:
- 架构
tags:
- HTTP安全
- restfull
---

> 如果启用HTTPS，基本上就不用考虑传输的安全问题，只需要考虑身份认证问题

### 一、开始之前
&emsp;&emsp;在平时的工作中，经常会碰到跟第三方系统做集成的需求，常用的技术手段有webservice，现在要给大家分享的是如何设计一个安全的可供远程访问API。  
提供两种调用方式：  
1.前端ajax调用（当然前提是解决了跨域问题）  
2.后端发起HTTP请求调用

### 二、整体结构

![整体结构](/images/20160815/RemoteAPI1.jpg)
#### 简要说明
- 服务器端提供给请求端key与secret密钥，这里可以增加IP过滤，如果带上key请求的请求端IP跟服务器上配置的IP不相符则返回401
- 请求端拿到key与secret密钥后，发起请求时，直接把key放到header中，然后用数据摘要算法（如md5，sha512）对请求数据生成数字签名sign（其实就是一个字符串），一起放到header中
- 服务器端的第一个过滤器首先过滤远程API（可以考虑远程API都用api/前缀打头），在这里首先验证key在系统中有没有，可选验证IP；如果待用sign，则意味着带请求数据，这个时候用服务器存着的secret对数据进行数字签名，与客户端传过来的进行对比（防数据篡改），一切都通过后再把这个key对应的user放session上
- 顺利通过session登录验证过滤器
- URL ACL权限验证过滤器一般是控制哪些人、角色、群组能访问这个URL
- 所有权限验证通过后到action-service顺利取到数据并返回

> 总的来说，远程API访问在于开一个口供HTTP请求交互数据，核心在于`权限的控制`

### 三、安全设计

#### 防篡改
> 防篡改的意思为防止数据在网络传输过程中被恶意篡改，比如在客户端-服务器端之间有个代理服务器（客户端-代理服务器-服务器），这个时候就可以在代理服务器在对数据动手脚，雁过拔毛

&emsp;&emsp;数据防篡改的核心原理在于对传输的数据进行数字签名。客户端发起请求钱，用服务器端拿到的密钥对数据进行前面，然后把签名与数据一起发送给服务器端，服务器端再用自己的密钥对收到的数据进行数字签名，与传过来的签名对比是否一致  
下面是一些相关java代码：  
##### 1. 服务器端生成密钥并保存到数据库中  

```java
/**
 * 产生HmacSHA512摘要算法的密钥
 * 密钥可以为任意字符串，使用它来生成密钥可以获得更高的安全性
 */  
public byte[] initHmacSHA512Key() throws NoSuchAlgorithmException {  
    // 初始化HmacMD5摘要算法的密钥产生器  
    KeyGenerator generator = KeyGenerator.getInstance("HmacSHA512");  
    // 产生密钥  
    SecretKey secretKey = generator.generateKey();  
    // 获得密钥  
    byte[] key = secretKey.getEncoded();  
    return key;  
}

//把byte数组转为为字符串方便存入数据库
String secret = new BigInteger(1, initHmacSHA512Key()).toString(16);
```

##### 2. 客户端/服务器端对传输的数据计算摘要

```java
//还原密钥字符串为byte数组
byte[] secret = new BigInteger(config.get(key).getConnectSecret(),16).toByteArray();
String sign = Tools.encodeHmacSHA512(postData.getBytes(), secret);

/**
 * HmacSHA512摘要算法
 * 对于给定生成的不同密钥，得到的摘要消息会不同，所以在实际应用中，要保存我们的密钥
 */  
public static String encodeHmacSHA512(byte[] data, byte[] key) throws Exception {  
    // 还原密钥  
    SecretKey secretKey = new SecretKeySpec(key, "HmacSHA512");  
    // 实例化Mac  
    Mac mac = Mac.getInstance(secretKey.getAlgorithm());  
    //初始化mac  
    mac.init(secretKey);  
    //执行消息摘要  
    byte[] digest = mac.doFinal(data);  
    return new HexBinaryAdapter().marshal(digest);//转为十六进制的字符串  
}
```

> 其实这里的密钥可以为任意字符串，这里之所以用生成的方式原因为这样长度更长，更安全，不易破解



#### 防重放

> 防重放的意思这样的，假如请求链为客户端-代理-服务器，那么不法分子可以在代理上把整个整个请求拷贝下来，然后再很遥远的未来用拷贝下来的请求重新请求服务器获取数据。很多时候我们是通过cookie信息来让服务器确认客户端的身份，这种情况就相当于走了一趟别人的通道，然后身份被窃取了。

&emsp;&emsp;有两种方式解决重放攻击问题，一种为随机数办法，第二种为TOTP算法（google的动态验证码就使用这种算法，另外还有很多银行的动态密钥）

##### 1.随机数办法
> 《HTTP权威指南》中的13章讲得很清楚，可以看看这章。直接从书中截个图开始讲

![请求逻辑](/images/20160815/RemoteAPI2.jpg)  
&emsp;&emsp;图中的(a)部分为未改进方式，(b)为性能改进方式。从图a中可以看出，每次`真正请求`前都必须发起一次质询请求拿到授权授权码（随机数），然后再发起真正请求，在真正请求中带上这个授权码，服务器接收到请求后先验证授权码是否存在，不存在则意味着不合法，如果存在，则删除旧的授权码。`相当每次请求都必须带上授权码，且一个授权码只能使用一次`。图(b)是做的一个改进，不用每次请求前都请求一次，只需要请求一次即可，每次正式请求成功后都返回下一次的授权码。

> 实际中的问题：需要考虑http请求的并发特性，在并发情况下妥善生成与删除授权码

#### 2.TOTP算法
&emsp;&emsp;这种算法跟google的动态验证码一样，客户端与服务器端通过相同的密钥与时间生成授权码（在一段时间内有效，需保证服务器端与客户端无时间差异，或差异很小）  
算法说明：  
> google动态验证码原理：服务器端通过特定算法随机生成一个密钥，并且把这个密钥保存在数据库中。根据生成的密钥在页面上生成一个二维码，内容是一个 URI 地址。用户通过扫描二维码，把密钥保存在客户端。 客户端每 30 秒使用密钥和时间戳通过一种算法生成一个 6 位数字的一次性 密码。服务器端使用保存在数据库中的密钥和时间戳通过同一种算法生成一个 6 位数字的一次性密码， 用户登陆时输入一次性密码与服务器端进行验证，如果一 样，就登录成功了。  
资料：  
<a href="https://www.zhihu.com/question/20462696">谷歌验证 (Google Authenticator) 的实现原理是什么？</a>
