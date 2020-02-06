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
# DELETE /jz-fe-http-log
# 查看索引相关信息
GET nginx-log-bjdaojiacom-2019.07.20
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
      "url":{
        "type" : "text",
        "fields" : {
          "keyword" : {
            "type" : "keyword"
          }
        }
      },
      "method":{"type":"keyword"},
      "param":{"type":"text"},
      "response":{"type":"text"},
      "message":{
        "type":"text",
        "fields" : {
          "keyword" : {
            "type" : "keyword"
          }
      }},
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

# 更新索引-修改字段
PUT jz-fe-http-log/_mapping
{
    "properties": {
      "url":{
        "type":"text",
        "fields" : {
          "keyword" : {
            "type" : "keyword"
          }
      }}
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
# 删除指定查询数据 - 前缀匹配
# POST /jz-fe-http-log/_delete_by_query
POST /jz-fe-http-log/_delete_by_query
{
  "size":5,
   "query": {
        "regexp": { "projectName": "daily-clean-v2.+" }
    }
}

POST /jz-fe-http-log/_delete_by_query
{
  "size":5,
   "query": {
        "match": { "projectName": "<%- 909275197+844400363 %>" }
    }
}

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
  "query": {
      "bool": {
          "must": [
              {
                "range": {
                  "serverDate": {
                    "gte": "2019-10-30 14:57:00",
                    "lte": "2019-10-31 14:57:00"
                  }
                }
              }
          ]
      }
  },
  "aggs": {
    "group_by_projectName": {
      "terms": {
        "size": 50,
        "field": "projectName"
      }
    }
  }
}

# 按照git group+项目名称分组
GET jz-fe-http-log/_search
{
  "size":0,
  "aggs": {
    "group_by_gitGroup": {
      "terms": {
        "size": 50,
        "field": "gitGroup"
      },
      "aggs": {
        "group_by_projectName": {
        "terms": {
          "size": 50,
          "field": "projectName"
        }
      }
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

# 最近一周所有问题增长趋势
GET jz-fe-http-log/_search
{
  "size":0,
  "track_total_hits": 100000,
  "query": {
        "bool": {
            "must": [
                {
                  "range": {
                    "serverDate": {
                      "gte": "2019-10-30 14:57:00",
                      "lte": "2019-10-31 14:57:00"
                    }
                  }
                }
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

# 上一周指定项目的问题增长趋势
GET jz-fe-http-log/_search
{
  "size":1,
  "query": {
    "bool": {
      "must": [
        { "match": { "projectName": "daily-clean" }},
        {
          "range": {
            "serverDate": {
              "gte": "2019-10-30 14:57:00",
              "lte": "2019-10-31 14:57:00"
            }
          }
        }
      ]
    }
  },
  "aggs": {
    "grouy_by_hour": {
      "date_histogram": {
        "field": "serverDate",
        "min_doc_count": 0,
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
        "size": 25,
        "field": "message.keyword"
      }
    }
  }
}

# 按指定项目中的message分页查询
POST /jz-fe-http-log/_search
{
  "size":2,
  "from": 0,
   "query": {
        "bool": {
            "must": [
                { "match": { "projectName": "daily-clean" }},
                { "match": { "message.keyword":  "无可用商家" }}
            ]
        }
    }
}

# 指定项目+message，按url分组，查询出指定错误的url排行
POST /jz-fe-http-log/_search
{
  "size":0,
  "from": 0,
  "query": {
    "bool": {
        "must": [
            { "match": { "projectName": "daily-clean" }},
            { "match": { "message.keyword": "timeout of 5000ms exceeded" }}
        ]
    }
  },
  "aggs": {
    "group_by_url": {
      "terms": {
        "field": "url.keyword"
      }
    }
  }
}


# 按时间降序
POST /jz-fe-http-log/_search
{
  "size":1,
   "query": {
        "bool": {
            "must": [
                { "match": { "projectName": "daily-clean" }},
                { "match": { "message.keyword":  "无可用商家" }}
            ]
        }
    },
    "sort": [
        {"serverDate": "desc"} ,
        {"_id": "asc"}    
    ]
}

# 按时间降序-searchAfter
POST /jz-fe-http-log/_search
{
  "size":1,
   "query": {
        "bool": {
            "must": [
                { "match": { "projectName": "daily-clean" }},
                { "match": { "message.keyword":  "无可用商家" }}
            ]
        }
    },
    "search_after":[
          1569489763000,
          "NP4pa20BFerq1YvUydLx"
        ],
    "sort": [
        {"serverDate": "desc"} ,
        {"_id": "asc"}    
    ]
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

# 查询最近一小时的数据，按项目分组
GET jz-fe-http-log/_search
{
  "size":0,
  "query": {
    "bool": {
      "must": [
        {
          "range": {
            "serverDate": {
              "gte": "2019-10-31 12:57:00",
              "lte": "2019-10-31 13:57:00"
            }
          }
        }
      ]
    }
  },
  "aggs": {
    "grouy_by_project": {
      "terms": {
        "size": 100,
        "field": "projectName"
      }
    }
  }
}



# 异常查询
GET jz-fe-http-log/_search
{
  "size":30,
  "query": {
    "bool": {
      "must": [

        {
          "range": {
            "serverDate": {
              "gte": "2020-01-18 17:00:00",
              "lte": "2020-01-18 17:30:00"
            }
          }
        }
      ]
    }
  },
  "aggs": {
    "grouy_by_project": {
      "terms": {
        "size": 100,
        "field": "projectName"
      }
    }
  }
}

GET jz-fe-http-log/_search
{
  "size":0,
  "query": {
    "bool": {
      "must": [
        { "match": { "projectName": "daily-clean-v2" }},
        {
          "range": {
            "serverDate": {
              "gte": "2020-01-18 07:00:00",
              "lte": "2020-01-18 13:00:00"
            }
          }
        }
      ]
    }
  },
  "aggs": {
    "grouy_by_message": {
      "terms": {
        "size": 25,
        "field": "message.keyword"
      }
    }
  }
}

GET jz-fe-http-log/_search
{
  "size":0,
  "query": {
    "bool": {
      "must": [
        { "match": { "projectName": "daily-clean-v2" }},
        {
          "range": {
            "serverDate": {
              "gte": "2020-01-18 07:00:00",
              "lte": "2020-01-18 13:00:00"
            }
          }
        }
      ]
    }
  },
  "aggs": {
    "group_by_url": {
      "terms": {
        "field": "url.keyword"
      }
    }
  }
}




# ----------------------------------------------------------------------------
#                                 性能监控相关
# ----------------------------------------------------------------------------
# 查看索引相关信息
GET jz-fe-performance-log
# 查看索引的文档总数
GET jz-fe-performance-log/_count
# 删除索引
DELETE /jz-fe-performance-log
# 性能监控-创建索引，禁止自动添加类型字段
PUT /jz-fe-performance-log
{
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

# 查询索引下的所有数据
GET /jz-fe-performance-log/_search
{
  "query": {
    "match_all": {}
  }
}

# 查看索引详情
GET /_cat/indices/jz-fe*?v&s=index
# 删除后释放空间
POST jz-fe-performance-log/_forcemerge
# 删除指定时间段的数据
POST jz-fe-performance-log/_delete_by_query?scroll_size=10000&slices=10
{
   "query": {
    "bool": {
      "must": [
        {
          "range": {
            "serverDate": {
              "lte": "2020-01-01 00:00:00"
            }
          }
        }
      ]
    }
  }
}

# 查询出指定时间段项目的平均打开时间
GET jz-fe-performance-log/_search
{
  "size":0,
  "track_total_hits": 10000000,
  "query": {
    "bool": {
      "must": [
        {
          "range": {
            "serverDate": {
              "gte": "2020-01-10 12:00:00",
              "lte": "2020-01-18 13:00:00"
            }
          }
        }
      ]
    }
  },
  "aggs":{
		"projectAvgTime":{
			"terms":{
			  "size": 50,
				"script": "'【'+doc['groupName'].value+'】'+doc['projectName'].value",
				"order": {
				  "avgTime": "desc"
				}
			},
			"aggs":{
				"avgTime":{
					"avg":{
						"field":"TTFLTime"
					}
				},
				"minTime":{
					"min":{
						"field":"TTFLTime"
					}
				},
				"maxTime":{
					"max":{
						"field":"TTFLTime"
					}
				}
			}
		}
	}
}

# 秒开率计算
POST jz-fe-performance-log/_search
{
  "size": 0,
  "track_total_hits": 10000000,
  "query": {
    "bool": {
      "must": [
        { "range": { "TTFLTime": { "lte": 3000 } } },
        { "match": { "projectName": "daily-clean-v2" }},
        {
          "range": {
            "serverDate": {
              "gte": "2020-01-10 12:00:00",
              "lte": "2020-01-18 13:00:00"
            }
          }
        }
      ]
    }
  }
}

# 平均首字节时间
GET jz-fe-performance-log/_search
{
  "size":0,
  "track_total_hits": 10000000,
  "query": {
    "bool": {
      "must": [
        {
          "range": {
            "serverDate": {
              "gte": "2020-01-10 12:00:00",
              "lte": "2020-01-18 13:00:00"
            }
          }
        }
      ]
    }
  },
  "aggs":{
		"avgTime":{
			"avg":{
				"field":"TTFLTime"
			}
		},
		"minTime":{
			"min":{
				"field":"TTFLTime"
			}
		},
		"maxTime":{
			"max":{
				"field":"TTFLTime"
			}
		}
	}
}

# 性能柱状图
GET jz-fe-performance-log/_search
{
  "size":0,
  "track_total_hits": 10000000,
  "query": {
    "bool": {
      "must": [
        { "range": { "TTFLTime": { "gte": 250, "lte": 349 } } },
        {
          "range": {
            "serverDate": {
              "gte": "2020-02-05 16:19:19",
              "lte": "2020-02-06 16:19:19"
            }
          }
        }
      ]
    }
  },
  "aggs":{
		"chartData":{
			"terms":{
			  "size": 1000,
			  "field": "TTFLTime",
				"script": "Math.round(_value/100)",
				"order": {
				  "_key": "asc"
				}
			}
		}
	}
}


```
