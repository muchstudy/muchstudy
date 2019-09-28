---
title: Elasticsearch常用增删改查记录
date: 2019-09-28 23:25:38
tags:
---

最近使用Elasticsearch来做异常监控的存储，写了不少ES的索引操作，以及对数据的增删改查操作，记录一下，以备不时之需。

```shell
# 列出所有索引
GET _cat/indices

# 查看indices
GET /_cat/indices/jz-fe*?v&s=index

# 查看索引的文档总数
GET jz-fe-http-log/_count

# 删除索引
DELETE /jz-fe-http-log
# 查看索引相关信息
GET jz-fe-http-log
# 创建索引，禁止自动添加类型字段
PUT /jz-fe-http-log
{
  "mappings": {
    "dynamic": "strict",
    "properties":{
      "gitGroup":{"type":"keyword"},
      "projectName":{"type":"keyword"},
      "level":{"type":"keyword"},
      "code":{"type":"keyword"},
      "href":{"type":"text"},
      "url":{"type":"text"},
      "method":{"type":"keyword"},
      "param":{"type":"text"},
      "response":{"type":"text"},
      "message":{"type":"text"},
      "stack":{"type":"text"},
      "clientDate":{"type":"date","format":"yyyy-MM-dd HH:mm:ss"},
      "serverDate":{"type":"date","format":"yyyy-MM-dd HH:mm:ss"},
      "uid": {"type":"keyword"},
      "phone": {"type":"keyword"},
      "os":{"type":"text"},
      "platform":{"type":"text"},
      "browser":{"type":"keyword"},
      "version":{"type":"keyword"},
      "userAgent":{"type":"text"},
      "status":{"type":"integer"},
      "closeDate":{"type":"date","format":"yyyy-MM-dd HH:mm:ss"}
    }
  }
}

# 更新索引-增加字段
PUT jz-fe-http-log/_mapping
{
    "dynamic": "strict",
    "properties": {
      "status":{"type":"integer"},
      "closeDate":{"type":"date","format":"yyyy-MM-dd HH:mm:ss"}
    }
}

# 增加数据
POST /jz-fe-http-log/_doc
{
  "os":"OS X",
  "platform":"iPhone",
  "browser":"Safari",
  "version":"11.0",
  "clientDate":"2019-08-23 18:54:03"
}

# 查询索引下的所有数据
GET /jz-fe-http-log/_search
{
  "query": {
    "match_all": {}
  }
}

# 指定key查询
GET /jz-fe-http-log/_search
{
  "query": {
    "match": {
      "_id": "BYG_SG0BW-qMNGXgwzPg"
    }
  }
}

# 按文档ID更新指定字段数据
POST /jz-fe-http-log/_doc/R9Wm_mwBAl5tA0U3Hq4l/_update
{
   "doc" : {
      "status": 1
   }
}

# 按id删除数据
DELETE /jz-fe-http-log/_doc/l0_772wBAl5tA0U3dpBH


# 异常总数
GET /jz-fe-http-log/_search
{
  "size":0,
  "query": {
    "match_all": {}
  }
}

# 已处理异常总数
GET /jz-fe-http-log/_search
{
  "size":0,
  "query": {
    "match": {
      "status": 1
    }
  }
}

# 当天异常总数
GET /jz-fe-http-log/_search
{
  "size":0,
  "query": {
    "match": {
      "serverDate": "now/1d"
    }
  }
}

# 今日异常数
POST /jz-fe-http-log/_search
{
  "size":0,
   "query": {
        "bool": {
            "must": [
                { "match": { "serverDate": "now/1d" }},
                { "match": { "status":  "1" }}
            ]
        }
    }
}


# 按照项目名称分组
GET jz-fe-http-log/_search
{
  "size":0,
  "aggs": {
    "group_by_projectName": {
      "terms": {
        "field": "projectName"
      }
    }
  }
}

# 指定项目下，错误等级分组
GET jz-fe-http-log/_search
{
  "size":0,
  "query": {
    "match": {
      "projectName": "daily-clean"
    }
  },
  "aggs": {
    "group_by_projectName": {
      "terms": {
        "field": "level"
      }
    }
  }
}

# 按天分组、计数
GET jz-fe-http-log/_search
{
  "size":0,
  "aggs": {
    "grouy_by_day": {
      "date_histogram": {
        "field": "serverDate",
        "interval": "day",
         "format" : "yyyy-MM-dd"
      }
    }
  }
}

# 最近一周问题增长趋势
GET jz-fe-http-log/_search
{
  "size":0,
  "query": {
        "bool": {
            "must": [
                { "match": { "serverDate": "now-1w/w" }}
            ]
        }
    },
  "aggs": {
    "grouy_by_hour": {
      "date_histogram": {
        "field": "serverDate",
        "interval": "hour",
         "format" : "yyyy-MM-dd HH:mm:ss"
      }
    }
  }
}

# 指定项目，按message分组
GET jz-fe-http-log/_search
{
  "size":0,
  "query": {
        "bool": {
            "must": [
                { "match": { "projectName": "csworker" }}
            ]
        }
    },
  "aggs": {
    "grouy_by_message": {
      "terms": {
        "field": "message.keyword"
      }
    }
  }
}

# 指定时间段，按项目分组
GET jz-fe-http-log/_search
{
  "size":0,
  "query": {
        "range": {
            "serverDate": {
              "gt": "2019-09-24 09:00:00",
              "lt": "2019-09-24 10:00:00",
              "format": "yyyy-MM-dd HH:mm:ss"
            }
        }
    },
  "aggs": {
    "grouy_by_project": {
      "terms": {
        "field": "projectName"
      }
    }
  }
}


```
