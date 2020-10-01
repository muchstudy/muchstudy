---
title: Elasticsearch索引自动化管理
date: 2020-06-04 18:19:11
tags:
---
{% asset_img 索引.jpg %}

## 背景
前端性能监控的日志之前为单一索引，随着日志内容的不断增多，索引文件变得越来越多大（官方建议单个索引文件不要超过20G）。

在此种方案下只能定时通过`delete query`的方式删除xxx天之前的数据，此种方式删除数据时异常缓慢，而且磁盘空间不会立即释放。

亟需采取新的索引方案解决该问题，比如按天生成索引，定时删除一个月之前的索引文件,直接删除索引文件的效率会高不少。

## 索引创建

### 索引模板

索引模板是为了方便按天去生成相同配置的索引文件，样例如下：

```
# 创建索引模板
PUT _template/jz-fe-performance-log-template
{
  "index_patterns" : ["jz-fe-performance-log-*"],
  "settings" : {
      "index" : {
        "lifecycle" : {
          "name" : "jz-fe-log-15days"
        }
      }
    },
  "mappings": {
    "dynamic": "strict",
    "properties":{
      "groupName":{"type":"keyword"},
      "projectName":{"type":"keyword"},
      "href":{"type":"keyword"},
      "clientDate":{"type":"date","format":"yyyy-MM-dd HH:mm:ss"},
      "serverDate":{"type":"date","format":"yyyy-MM-dd HH:mm:ss"},
      "appId":{"type":"keyword"},
      "hmsr":{"type":"keyword"},
      "znsr":{"type":"keyword"},
      "hmpl":{"type":"keyword"},
      "unloadTime":{"type":"integer"},
      "redirectTime":{"type":"integer"},
      "appCacheTime":{"type":"integer"},
      "dnsTime":{"type":"integer"},
      "tcpTime":{"type":"integer"},
      "requestTime":{"type":"integer"},
      "responseTime":{"type":"integer"},
      "analysisTime":{"type":"integer"},
      "loadEventTime":{"type":"integer"},
      "connectTime":{"type":"integer"},
      "resourceTime":{"type":"integer"},
      "domReadyTime":{"type":"integer"},
      "TTFBTime":{"type":"integer"},
      "TTSRTime":{"type":"integer"},
      "TTDCTime":{"type":"integer"},
      "TTFLTime":{"type":"integer"},
      "FMPTime":{"type":"integer"},
      "uid":{"type":"keyword"},
      "phone":{"type":"keyword"},
      "platform":{"type":"keyword"},
      "os":{"type":"keyword"},
      "browser":{"type":"keyword"},
      "version":{"type":"keyword"},
      "userAgent":{"type":"text"},
      "ip":{"type":"keyword"},
      "networkType":{"type":"keyword"},
      "ISP":{"type":"keyword"},
      "region":{"type":"keyword"}
    }
  }
}
```

说明：

- `PUT _template/jz-fe-performance-log-template`创建名称为`jz-fe-performance-log-template`的索引模板
- `index_patterns`中的`jz-fe-performance-log-*`代表索引名称为`jz-fe-performance-log-`的索引都按这个模板的配置去生成
- `settings`中的`lifecycle`项注明以哪一个lifecycle配置来管理该索引，在后续的索引删除部分会使用到
- `mappings`就是所有文档的配置了，`strict`表面为严格模式，索引数据只能为下面声明的字段名称，否则无法保存

### 数据保存

在数据调用Node服务接口时，做如下处理，即可按天把日志保存到当天的日志文件中
```javascript

const { Client } = require('@elastic/elasticsearch')

// 性能数据索引
const PERFORMANCE_PROD_INDEX = 'jz-fe-performance-log'
const PERFORMANCE_TEST_INDEX = 'jz-fe-performance-log-test'

/**
 * ES数据插入操作
 * @param index 索引
 * @param data 数据
 * @returns {Promise<ApiResponse<any>>}
 */
async function base ({ index, data }) {
    const res = await client.index({
        index: index,
        body: data
    }).catch(err=>{
        console.error('err', JSON.stringify(err))
    })
    return res
}

/**
  * 日期格式化
  * 返回 2020-6-5类型数据
  * @param {日期} date
  */
function dateFormat (date) {
    const year = date.getFullYear()
    const month = date.getMonth() + 1
    const day = date.getDate()
    return year + '-' + month + '-' + day
}

/**
 * 性能数据入库
 * @param body
 * @returns {Promise<ApiResponse<any>|TResult>}
 */
async function performanceAdd (data) {
    const index = tools.isTestENV() ? PERFORMANCE_TEST_INDEX : `${PERFORMANCE_PROD_INDEX}-${dateFormat(new Date())}`
    const res = await base({ index: index, data })
    return res
}

```

说明：
- 索引名称以`jz-fe-performance-log`打头的索引文件没有时都为以上面的模板新创建
- 性能数据保存时，指定保存到当天的索引文件中

## 索引删除

实现了索引文件按天拆分之后，下一步就需要考虑如何把索引文件管理起来。

日志数据一般只保留一个月，这个时候可以考虑可以写一个程序定时去删除一个月之前的索引。

Elasticsearch 6.6开始提供了一个叫`Index Lifecycle Management`的功能来管理日志。

### ILM配置
可以通过kibana可视化的做配置，也可以通过写ES语句的方式创建

{% asset_img ES-IML-1.jpg %}

点击`Create policy`即可创建对应配置

{% asset_img ES-IML-2.jpg %}

可通过如下方式查看刚才创建的具体配置信息
```
# 查看Index Lifecycle Manegment配置
GET /_ilm/policy/jz-fe-log-15days
```

返回结果如下
```
{
  "jz-fe-log-15days" : {
    "version" : 1,
    "modified_date" : "2020-06-04T10:02:16.233Z",
    "policy" : {
      "phases" : {
        "hot" : {
          "min_age" : "0ms",
          "actions" : {
            "set_priority" : {
              "priority" : 100
            }
          }
        },
        "delete" : {
          "min_age" : "15d",
          "actions" : {
            "delete" : { }
          }
        }
      }
    }
  }
}
```

说明：
- 索引要被哪个ILM管理是在索引的`settings`的`lifecycle`处指定的
- ES默认10分钟执行一次检查，如果对应索引满足创建时间大于15天，则删除索引

## 其他操作

下面是测试验证该功能会用到的相关ES语句
```shell
# =========索引模板相关===========

# 创建索引模板
PUT _template/jz-fe-test-template
{
  "index_patterns" : ["jz-fe-test-*"],
  "settings" : {
      "index" : {
        "lifecycle" : {
          "name" : "jz-fe-log-30s"
        }
      }
    },
  "mappings": {
    "dynamic": "strict",
    "properties":{
      "gitGroup":{"type":"keyword"},
      "projectName":{"type":"keyword"}
    }
  }
}


# 查看指定template信息
GET /_template/jz-fe-test-template
# 查看指定前缀template信息
GET /_template/jz-fe-*

# 删除索引模板
DELETE /_template/jz-fe-test-template

# 增加数据
POST /jz-fe-test-2020-06-03/_doc
{
  "gitGroup":"OS X",
  "projectName":"iPhone2"
}

# 查询数据
GET jz-fe-test-2020-06-*/_search

# 查看索引详情
GET /_cat/indices/jz-fe-test*?v&s=index

# 删除测试索引
DELETE /jz-fe-test-2020-06-*

# =========Index Lifecyce Management===========

# 设置10秒刷新1次(即定时器间隔)，生产环境10分种刷新一次
# 可设置索引的保留时间为30s，每10s判断一次是否满足条件
PUT _cluster/settings
{
  "persistent": {
    "indices.lifecycle.poll_interval":"10s"
  }
}

# 查看设置
GET _cluster/settings

# 查看Index Lifecycle Manegment配置
GET /_ilm/policy/jz-fe-log-15days

```

<div style="width:70%;margin:auto">
<img src='http://muchstudy.com/2020/04/04/%E8%81%8A%E8%81%8A%E4%B8%80%E7%BA%BF%E5%BC%80%E5%8F%91%E7%9A%84%E5%9F%BA%E6%9C%AC%E7%B4%A0%E5%85%BB/%E5%85%AC%E4%BC%97%E5%8F%B7%E4%BA%8C%E7%BB%B4%E7%A0%81.gif'>
</div>
