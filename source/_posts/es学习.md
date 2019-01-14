---
title: es学习
date: 2018-11-29 13:58:32
tags:
---

 ### Elasticsearch是什么？

Elastic Search 是一个基于Lucene构建的开源，分布式搜索引擎。设计用于云计算中，实时搜索，稳定，可靠，快速，安装使用方便。支持通过HTTP使用JSON进行数据索引。

* 分布式的实时文件存储，每个字段都被索引并可被搜索

* 实时分析的分布式搜索引擎

* 可以扩展到上百台服务器，处理PB级结构化或非结构化数据

<!--more-->
![第一个测试图片](es.png)


### 安装head插件

* 从github下载head插件源码
``` bash
git clone https://github.com/mobz/elasticsearch-head.git
cd elasticsearch-head/
npm install
```

* 修改elasticsearch 配置文件 /opt/elasticsearch-5.5.1/config/elasticsearch.yml 增加跨域的配置(需要重启es才能生效)
``` bash
http.cors.enabled: true
http.cors.allow-origin: "*"
```
* 编辑/opt/elasticsearch-head/Gruntfile.js，修改服务器监听地址，增加hostname属性，将其值设置为*。

``` bash
    connect: {
                server: {
                        options: {
                                hostname: '*',
                                port: 9100,
                                base: '.',
                                keepalive: true
                        }
                }
        }
```
* 编辑elasticsearch-head/_site/app.js，修改head连接es的地址，将localhost修改为es的IP地址
``` bash
# 原配置
this.base_uri = this.config.base_uri || this.prefs.get("app-base_uri") || "http://localhost:9200";
# 将localhost修改为ES的IP地址
this.base_uri = this.config.base_uri || this.prefs.get("app-base_uri") || "http://YOUR-ES-IP:9200";
``` 
* 启动elasticsearch-head

``` bash
cd /opt/elasticsearch-head/ && ./node_modules/grunt/bin/grunt server
```

* 启动es命令
``` bash
cd /opt/elasticsearch-5.5.1/bin/ && ./elasticsearch -d
```

## es restapi

* 创建索引
``` bash
方式一 curl -XPUT http://localhost:9200/es_test?
方式二 curl -X PUT 'localhost:9200/weather'

返回json
{"acknowledged":true,"shards_acknowledged":true} 代表成功
```

* 删除索引

``` bash
curl -X DELETE 'localhost:9200/weather'
```
## 中文分词设置 安装ik插件

``` bash
/opt/elasticsearch-5.5.1/bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v5.5.1/elasticsearch-analysis-ik-5.5.1.zip
```
接着，重新启动 Elastic，就会自动加载这个新安装的插件。


``` bash

{
  "mappings": {
    "person": {
      "properties": {
        "user": {
          "type": "text",
          "analyzer": "ik_max_word",
          "search_analyzer": "ik_max_word"
        },
        "title": {
          "type": "text",
          "analyzer": "ik_max_word",
          "search_analyzer": "ik_max_word"
        },
        "desc": {
          "type": "text",
          "analyzer": "ik_max_word",
          "search_analyzer": "ik_max_word"
        }
      }
    }
  }
}

```
上面代码中，首先新建一个名称为accounts的 Index，里面有一个名称为person的 Type。person有三个字段。
``` bash
user
title
desc
```
这三个字段都是中文，而且类型都是文本（text），所以需要指定中文分词器，不能使用默认的英文分词器。

Elastic 的分词器称为 analyzer。我们对每个字段指定分词器。

``` bash
"user": {
  "type": "text",
  "analyzer": "ik_max_word",
  "search_analyzer": "ik_max_word"
}
```
上面代码中，analyzer是字段文本的分词器，search_analyzer是搜索词的分词器。ik_max_word分词器是插件ik提供的，可以对文本进行最大数量的分词。

## 数据操作

* 新增记录
``` bash

http://localhost:9200/accounts/person/3  put

{
  "user": "曹子龙",
  "title": "java工程师",
  "desc": "百分点"
}

# 返回信息

{
"_index": "accounts",
"_type": "person",
"_id": "3",
"_version": 1,
"result": "created",
"_shards": {
"total": 2,
"successful": 1,
"failed": 0
},
"created": true
}
```

* 查看记录

``` bash
http://localhost:9200/accounts/person/3?pretty=true/

# 返回信息
{
"_index": "accounts",
"_type": "person",
"_id": "3",
"_version": 1,
"found": true,
"_source": {
"user": "曹子龙",
"title": "java工程师",
"desc": "百分点"
}
}

# 如果id找不到
{
"_index": "accounts",
"_type": "person",
"_id": "311",
"found": false
}
```
* 删除记录

``` bash
http://localhost:9200/accounts/person/3 delete

#返回信息
{
"found": true,
"_index": "accounts",
"_type": "person",
"_id": "3",
"_version": 2,
"result": "deleted",
"_shards": {
"total": 2,
"successful": 1,
"failed": 0
}
}
# 如果已删除 或未找到
{
"found": false,
"_index": "accounts",
"_type": "person",
"_id": "3",
"_version": 3,
"result": "not_found",
"_shards": {
"total": 2,
"successful": 1,
"failed": 0
}
}
```

* 更新记录信息
``` bash
http://localhost:9200/accounts/person/3/ put


{"user":"曹子龙","title":"java工程师","desc":"沈阳大数据"}
```

* 返回所有记录