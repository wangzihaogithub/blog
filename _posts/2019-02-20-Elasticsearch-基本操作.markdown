---
layout: post
title:  "Elasticsearch-基本操作"
tags: Elasticsearch
---

## 基本增删改查的操作（可以直接拷贝到kibana的开发工具中）

```
    # 保存
    PUT /wang-indexname/wang-indextype/id2
    {
        "name" : "wang",
        "password" :  "123",
        "age" :        100,
        "text" :      "测试wang ",
        "list": [ "a", "b" ]
    }
    
    # 保存 - 版本号处理冲突，当并发更新数据时防止更新丢失。
    PUT /wang-indexname/wang-indextype/id2?version=1
    {
        "name" : "zhang",
        "password" :  "123",
        "age" :        100,
        "text" :      "测试zhang ",
        "list": [ "c", "D" ]
    }
    
    # 保存 - 批量
    POST /wang-indexname/wang-indextype/_bulk
    { "index": { "_id": 1 }}
    { "price" : 10, "productID" : "XHDK-A-1293-#fJ3" }
    { "index": { "_id": 2 }}
    { "price" : 20, "productID" : "KDKE-B-9947-#kL5" }
    { "index": { "_id": 3 }}
    { "price" : 30, "productID" : "JODL-X-1937-#pV7" }
    { "index": { "_id": 4 }}
    { "price" : 30, "productID" : "QQPX-R-3956-#aD8" }
    
    # 局部更新 存在的字段覆盖，不存在的字段添加-版本失败重试5次
    POST /wang-indexname/wang-indextype/id1/_update?retry_on_conflict=5
    {
       "doc" : {
          "array" : [ "movie","music" ],
          "age": 26
       }
    }
    
    
    
    #查询id1的数据 
    GET /wang-indexname/wang-indextype/id1
    
    #查看查询细节
    GET /wang-indexname/wang-indextype/_validate/query?explain
    
    #查询所有数据，默认返回前十条
    GET /wang-indexname/wang-indextype/_search
    {
      "query": {
        "match_all": {}
      }
    }
    
    
    
    #删除
    DELETE /wang-indexname/wang-indextype/id1
    
    
    
    
    
    
    #查询集群节点是否禁用内存交换（swapping）
    GET _nodes?filter_path=**.mlockall
    
    #查询集群节点的最大文件描述符
    GET _nodes/stats/process?filter_path=**.max_file_descriptors
    
    #打开（默认关闭）动态数字映射
    PUT my_index_name
    
    GET _cat/shards
    
    
    GET _cluster/settings
    
    #查询各个节点安装的插件
    GET /_cat/plugins?v&s=component&h=name,component,version,description

    #查询分词结果
    POST test-ig_zues_db_ig_zues-biz_corp/_analyze
    {
      "field": "name",
      "text": "豪-大数据工程师"
    }
    
```
