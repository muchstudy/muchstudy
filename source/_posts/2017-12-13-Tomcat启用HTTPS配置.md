---
title: Tomcat启用HTTPS配置
date: 2017-12-13 17:26:41
categories:
- HTTP
---

## 获取证书

&emsp;&emsp;可使用jdk自带的keytool开生成证书，此种方式与向第三方权威机构购买的证书的区别为，第一次请求时需要选择信任站点并继续访问，在浏览器的地址框里会显示不安全的红色提醒。

&emsp;&emsp;使用`keytool -genkey -alias tomcat -validity 3650 -keyalg RSA -keystore D://.keystore`即可在D盘生成`.keystore`文件，如下所示
```
PS C:\Users\YiYing\Desktop> keytool -genkey -alias tomcat -keyalg RSA -keystore D://.keystore
输入密钥库口令:
再次输入新口令:
您的名字与姓氏是什么?
  [Unknown]:  YI
您的组织单位名称是什么?
  [Unknown]:  CSS
您的组织名称是什么?
  [Unknown]:  CSS
您所在的城市或区域名称是什么?
  [Unknown]:  BJ
您所在的省/市/自治区名称是什么?
  [Unknown]:  BJ
该单位的双字母国家/地区代码是什么?
  [Unknown]:  CHINA
CN=YI, OU=CSS, O=CSS, L=BJ, ST=BJ, C=CHINA是否正确?
  [否]:  Y

输入 <tomcat> 的密钥口令
        (如果和密钥库口令相同, 按回车):
PS C:\Users\YiYing\Desktop>
```
说明：
- -alias为别名
- -validity 3650有效期为10年
- -keystore为生成的文件路径

```html
Option Defaults
Below are the defaults for various option values.
-alias "mykey"

-keyalg
    "DSA" (when using -genkeypair)
    "DES" (when using -genseckey)

-keysize
    1024 (when using -genkeypair)
    56 (when using -genseckey and -keyalg is "DES")
    168 (when using -genseckey and -keyalg is "DESede")

-validity 90

-keystore the file named .keystore in the user's home directory

-storetype the value of the "keystore.type" property in the security properties file,
           which is returned by the static getDefaultType method in java.security.KeyStore

-file stdin if reading, stdout if writing

-protected false
```
> 文档地址：
https://docs.oracle.com/javase/6/docs/technotes/tools/solaris/keytool.html

## Tomcat配置

&emsp;&emsp;需要修改tomcat的server.xml文件，如下所示：
```xml
<!-- Define a SSL HTTP/1.1 Connector on port 8443
         This connector uses the BIO implementation that requires the JSSE
         style configuration. When using the APR/native implementation, the
         OpenSSL style configuration is required as described in the APR/native
         documentation -->

    <Connector SSLEnabled="true" clientAuth="false" keystoreFile="d://.keystore" keystorePass="111111" maxThreads="150" port="8443" protocol="org.apache.coyote.http11.Http11Protocol" scheme="https" secure="true" sslProtocol="TLS"/>
```
&emsp;&emsp;本来这一段是注释掉的，去掉注释并增加`keystoreFile`和`keystorePass`参数即可。

&emsp;&emsp;最后，启动服务器，使用`https://localhost:8443/`即可访问服务器。

>文档地址：
1. Tomcat7:https://tomcat.apache.org/tomcat-7.0-doc/ssl-howto.html
2. Tomcat8:https://tomcat.apache.org/tomcat-8.0-doc/ssl-howto.html
