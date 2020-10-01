---
title: 解决sql脚本导入Oracle重复生成check约束问题
subtitle: Import sql to oracle automatically add check constraint
date: 2016-10-08 16:06:38
categories:
- DBMS
---

&emsp;&emsp;之前一位细心的同事发现产品的全量sql脚本中有一些重复的check约束检查，就像下图这样的

{% asset_img 重复脚本.jpg 重复脚本 %}

怪异之处还在于，每次执行一遍该脚本，然后导出脚本，在导出脚本中重复的次数就会增加一遍。通过navicat，最终确认每导入一次就会新增加一条重复的check约束，如下图所示  

{% asset_img navicat.jpg navicat %}

&emsp;&emsp;这个全量脚本是直接从数据库中导出的，为了方便导入其他的Oracle数据库中，从产品的出货库导出时手动去掉了`服务名`、`双引号`。

&emsp;&emsp;通过如下步骤可复现该问题：  
**1.创建表**
```sql
CREATE TABLE PD_WEB_FILEUPLOAD_CHUNK (
ID VARCHAR2(32 BYTE) NOT NULL ,
MD5 VARCHAR2(32 BYTE) DEFAULT NULL  NULL ,
CHUNK NUMBER DEFAULT NULL  NULL ,
FILE_DIR VARCHAR2(200 BYTE) NOT NULL
)
```
可以看到上面的脚本中有`NOT NULL`的标识，执行完后在navicat中可以看到结果是这样的

{% asset_img navicatnew.jpg navicat %}

注意看，这里的check约束是带双引号的。


**2.执行增加check约束的脚本**

```sql
ALTER TABLE PD_WEB_FILEUPLOAD_CHUNK ADD CHECK (ID IS NOT NULL);
ALTER TABLE PD_WEB_FILEUPLOAD_CHUNK ADD CHECK (FILE_DIR IS NOT NULL);
```
执行了两遍后，结果如图所示

{% asset_img navicatThree.jpg navicat %}

测试到这里，以为最终终于找到了原因，确认为双引号的问题。抱着严谨的态度，再次确认了一下  

**3.执行带双引号的check约束的脚本**

```sql
ALTER TABLE PD_WEB_FILEUPLOAD_CHUNK ADD CHECK ("ID" IS NOT NULL);
ALTER TABLE PD_WEB_FILEUPLOAD_CHUNK ADD CHECK ("FILE_DIR" IS NOT NULL);
```
结果如图所示：
{% asset_img four.jpg navicat %}

靠！居然还是会重复生成！  

&emsp;&emsp;验证要这里，算是找出了原因。在全量导出的脚本中，创建表的脚本中已经隐含了检查约束，如果再显示的添加检查约束就会重复生成。所以，解决办法为需要手动删除所有显示的检查约束。
