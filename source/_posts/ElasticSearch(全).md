---
title: ElasticSearch(全)
date: 2024-04-26 00:00:00
author: Johns onLiam
tags: [ES]
top_img: https://wei-blog.oss-cn-beijing.aliyuncs.com/img/224634-171232839454f0.jpg
comments: true
---

# Elastic Stack

> `Elastic Stack`是一套构建在开源基础之上，可以让我们安全可靠地采集任何来源、任何格式的数据，并且实时地对数据进行搜索、分析和可视化工具链。

## ELK技术栈

从上面这段定义可以看出`Elastic Stack`的几个特点：采集、转换、搜索、分析、可视化，这些功能分别由`ElasticSearch`、`Kibana`、`Beats`、`Logstash`这几个组件来实现。

## 数据搜索

- 精准查询

查询数据表中name等于张三的数据 

select * from user where name = ‘张三’

- 模糊查询

查询数据表中name中包含张三

select * from user where name like ‘%张三%’

- 关联查询

查询数据表中任何列中可能包含张三的数据

select * from user where name like ‘%张三%’ or address like ‘%张三%’ or ........

- 搜索查询

想要查询任何列跟张三有关系的数据，张哥  三叔 张三哥哥

当你不确定查询条件时，我们会使用搜索

## 全文检索

全文检索是指：

- 通过一个程序扫描文本中的每一个单词，**针对单词建立索引，并保存该单词在文本中的位置、以及出现的次数**

- 用户查询时，通过之前建立好的索引来查询，**将索引中单词对应的文本位置、出现的次数返回给用户**，因为有了具体文本的位置，所以就可以将具体内容读取出来了

- 类似于通过字典中的检索**字表查字**的过程（想象一下百度搜索）

> - 通过一个程序扫描文本中的每一个单词，针对单词建立`倒排索引`，并保存该单词在文本中的位置、以及出现的次数
>
> ![image-20240427174659903](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20240427174659903.png)
>
> - 用户查询时，通过之前建立好的索引来查询，将索引中单词对应的文本位置、出现的次数返回给用户，因为有了具体文本的位置，所以就可以将具体内容读取出来了
>
> - 类似于通过字典中的检索字表查字的过程

## 搜索引擎

Lucene是一种高性能的全文检索库，在2000年开源，最初由大名鼎鼎的Doug Cutting（道格·卡丁）开发。Lucene不是一个完整的全文检索引擎，它只是提供一个基本的全文检索的架构。

## ElasticSearch

`Elasticsearch`是`Elastic Static`的核心是基于Lucene的搜索服务器，可以用`Elasticsearch`来存储我们的文档数据，利用Elasticsearch强大的搜索和分析功能为我们的网站提供支持，`ElasticSearch`是分布式搜索引擎和大数据实时分析引擎，可以实时分析计算数据并得出结果，还可以通过`Kibana`的各种图表将分析结果可视化。

基于Lucene的分布式全文检索引擎, 比较出名的两个 ：

- ElasticSearch

> - 海量数据存储和处理
>   - 大型分布式集群（数百台规模服务器）
>   - 处理PB级数据
>   - 小公司也可以进行单机部署
> - 开箱即用
>   - 简单易用，操作非常简单
>   - 快速部署生产环境
> - 作为传统数据库的补充
>   - 传统关系型数据库不擅长全文检索（MySQL自带的全文索引，与ES性能差距非常大）
>   - 传统关系型数据库无法支持搜索排名、海量数据存储、分析等功能
>   - Elasticsearch可以作为传统关系数据库的补充，提供RDBM无法提供的功能

- Solr

> - solr 单独纯粹做查询的效率高于 ES
> - 如果写入的频次和读取的频次都比较多, 采用ES的效率要高于Solr
> - ElasticSearch 和 Solr 都是基于Lucene开发的

# ES核心概念

## ES基本概念

![image-20230801120715040](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20230801120715040.png)

- 节点（Node）

运行了单个实例的ES主机称为节点，它是集群的一个成员，可以存储数据、参与集群索引及搜索操作。节点通过为其配置的ES集群名称确定其所要加入的集群。

- 集群（cluster**）**

ES可以作为一个独立的单个搜索服务器。不过，一般为了处理大型数据集，实现容错和高可用性，ES可以运行在许多互相合作的服务器上。这些服务器的集合称为集群。

> 一个集群会有多个节点组成，一个节点就是一个ES服务实例，通过配置cluster.name=‘’加入集群
>
> - node.master=ture（默认为ture）-候选主节点，只有成为候选主节点的实例才能参与投票选举
> - 主节点：负责索引的添加、删除，跟踪其它节点的正常运行，对数据分片的分配，收集集群中各节点的状态信息等
> - node.data=ture（默认为ture）数据节点：负责对数据的增、删、改、查、聚合等操作，数据的查询和存储，对机器的io cpu memory要求比较高，一般选择`高配置的机器作为数据节点`
> - 协调节点：其本身不是通过配置来设置，用户的请求会随机分发给一个节点，该节点就成为了协调节点，负责分发客户端的读写请求、收集结果
>
> `集群每个节点既可以是候选主节点也可以是数据节点`

- 分片（Shard）

ES的“分片(shard)”机制可将一个索引内部的数据分布地存储于多个节点，它通过将一个索引切分为多个底层物理的Lucene索引完成索引数据的分割存储功能，这每一个物理的Lucene索引称为一个分片(shard)。

这样的好处是可以把一个大的索引拆分成多个，分布到不同的节点上。降低单服务器的压力，构成分布式搜索，提高整体检索的效率（分片数的最优值与硬件参数和数据量大小有关）。`分片的数量只能在索引创建前指定`，并且索引创建后不能更改。

4）副本（Replica）

副本是一个分片的精确复制，每个分片可以有零个或多个副本。副本的作用一是提高系统的容错性，当某个节点某个分片损坏或丢失时可以从副本中恢复。二是提高es的查询效率，es会自动对搜索请求进行负载均衡。

`副本数不能大于节点数`

## ES数据架构

`Elasticsearch与数据库的类比`

`es7之后就没了type`

| 关系型数据库（比如Mysql） | 非关系型数据库（Elasticsearch） |
| ------------------------- | ------------------------------- |
| 数据库Database            | 索引Index                       |
| 表Table                   | 类型Type                        |
| 数据行Row                 | 文档Document                    |
| 数据列Column              | 字段Field                       |
| 约束 Schema               | 映射Mapping                     |

![image-20230801143242790](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20230801143242790.png)

- 索引（index）

ES将数据存储于一个或多个索引中，索引是具有类似特性的文档的集合。索引相当于SQL中的一个数据库。

- 类型（Type）

类型是索引内部的逻辑分区(category/partition)，然而其意义完全取决于用户需求。因此，一个索引内部可定义一个或多个类型(type)。类型相当于“表”。

- 文档（Document）

文档是Lucene索引和搜索的原子单位，它是包含了一个或多个域的容器，基于JSON格式进行表示。相当于mysql表中的row。

- 映射（Mapping）

映射是定义文档及其包含的字段如何存储和索引的过程。

![image-20230801143800853](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20230801143800853.png)

| 一级分类 | 二级分类     | 具体类型                             |
| -------- | ------------ | ------------------------------------ |
| 核心类型 | 字符串类型   | string,text,keyword                  |
| h        | 整数类型     | integer,long,short,byte              |
| h        | 浮点类型     | double,float,half_float,scaled_float |
| h        | 逻辑类型     | boolean                              |
| h        | 日期类型     | date                                 |
| h        | 范围类型     | range                                |
| h        | 二进制类型   | binary                               |
| 复合类型 | 数组类型     | array                                |
| f        | 对象类型     | object                               |
| f        | 嵌套类型     | nested                               |
| 地理类型 | 地理坐标类型 | geo_point                            |
| d        | 地理地图     | geo_shape                            |
| 特殊类型 | IP类型       | ip                                   |
| t        | 范围类型     | completion                           |
| t        | 令牌计数类型 | token_count                          |
| t        | 附件类型     | attachment                           |
| t        | 抽取类型     | percolator                           |

bit：比特

byte：一个字节=8bit   -2^7^ ~2^7^-1

short：两个字节=16bit   -2^15^ ~ 2^15^-1

integer：四个字节=32bit

long：八个字节=64bit

## ES集群架构

一个ES集群可以有多个节点构成，一个节点就是一个ES服务实例，通过配置`集群名称cluster.name`加入集群。

- 候选主节点：只有是候选主节点才可以参与选举投票，也只有候选主节点可以被选举为主节点。

- 主节点：负责索引的添加、删除，跟踪哪些节点是群集的一部分，对分片进行分配、收集集群中各节点的状态等，稳定的主节点对集群的健康是非常重要。

- 数据节点**：**负责对数据的增、删、改、查、聚合等操作，数据的查询和存储都是由数据节点负责，对机器的CPU，IO以及内存的要求比较高，一般选择高配置的机器作为数据节点。
- 协调节点：其本身不是通过设置来分配的，用户的请求可以随机发往任何一个节点，并由该节点负责分发请求、收集结果等操作，而不需要主节点转发。这种节点可称之为协调节点，集群中的任何节点都可以充当协调节点的角色。每个节点之间都会保持联系。

`集群中单个节点既可以是候选主节点也可以是数据节点`

# ES安装部署

> 参考自己整理过的文章：[ElasticSearch6.0.1及Azkaban3.8部署 ](https://liamjohnson-w.github.io/2023/10/17/2023.10.17/)

- 配置普通用户
  - 首先必须创建一个普通用户
  - 然后把es所在的目录所有者设置为这个普通用户


- 配置es环境

  - ![image-20230801151755388](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20230801151755388.png)

- 修改系统配置

  - >\$KAFKA_HOME/bin/kafka-server-start.sh \$KAFKA_HOME/config/server.properties 2>&1 >> $KAFKA_HOME/kafka-server.log &
    >
    >nohup /home/es/elasticsearch-7.10.2/bin/elasticsearch 2>&1 &
    >
    >nohup ：守护进程
    >
    >& : 后台运行
    >
    >2>&1 : 2-系统错误信息  1-系统信息 >&输出重定向
    >
    >\>> ：追加写入

  - bin/elasticsearch -d

- 安装可视化插件：elasticsearch-head

- 安装IK分词器

- 配置VSCode

# RestFul API

> REST，表示性状态转移（representation state transfer）。简单来说，就是用URI表示资源，用HTTP方法(GET, POST, PUT, DELETE)表征对这些资源的操作。
>
> URI：uniform resource indentify 统一资源标识符
>
> URL：uniform resource locate 统一资源定位符 是URI的一个子集

- 一个动作配置一个接口

![image-20230801155320880](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20230801155320880.png)

- 使用http方法类型判断执行的操作类型

GET - SELECT

POST - UPDATE

PUT - INSERT

DELETE - DELETE

## ES操作使用(Kibana)

- 创建索引

```json
PUT /my-index
{
    "mapping": {
        "properties": {
            "employee-id": {
                "type": "keyword",
                "index": false
            }
        }
    }
}
```

- 指定副本、分片数量

```json
// 创建指定分片数量、副本数量的索引
PUT /job_idx_shard
{
    "mappings": {
        "properties": {
            "employee-id": {
                "type": "keyword",
                "index": false
            }
        }
    },
    "settings": {
        "number_of_shards": 3,
        "number_of_replicas": 1
    }
}
```

- 字段类型

| 分类             | 类型名称                | 说明                                                         |
| ---------------- | ----------------------- | ------------------------------------------------------------ |
| 简单类型         | text                    | 需要进行全文检索的字段，通常使用text类型来对应邮件的正文、产品描述或者短文等非结构化文本数据。分词器先会将文本进行分词转换为词条列表。将来就可以基于词条来进行检索了。文本字段不能用于排序、也很少用于聚合计算。 |
|                  | keyword                 | 使用keyword来对应结构化的数据，如ID、电子邮件地址、主机名、状态代码、邮政编码或标签。可以使用keyword来进行排序或聚合计算。注意：keyword是不能进行分词的。 |
|                  | date                    | 保存格式化的日期数据，例如：2015-01-01或者2015/01/01 12:10:30。在Elasticsearch中，日期都将以字符串方式展示。可以给date指定格式：”format”: “yyyy-MM-dd HH:mm:ss” |
|                  | long/integer/short/byte | 64位整数/32位整数/16位整数/8位整数                           |
|                  | double/float/half_float | 64位双精度浮点/32位单精度浮点/16位半进度浮点                 |
|                  | boolean                 | “true”/”false”                                               |
|                  | ip                      | IPV4（192.168.1.110）/IPV6（192.168.0.0/16）                 |
| JSON分层嵌套类型 | object                  | 用于保存JSON对象                                             |
|                  | nested                  | 用于保存JSON数组                                             |
| 特殊类型         | geo_point               | 用于保存经纬度坐标                                           |
|                  | geo_shape               | 用于保存地图上的多边形坐标                                   |

- 创建测试索引

![image-20230802100140188](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20230802100140188.png)

```json
PUT /job_idx
{
    "mappings": {
        "properties" : {
            "area": { "type": "text", "store": true},
            "exp": { "type": "text", "store": true},
            "edu": { "type": "keyword", "store": true},
            "salary": { "type": "keyword", "store": true},
            "job_type": { "type": "keyword", "store": true},
            "cmp": { "type": "text", "store": true},
            "pv": { "type": "keyword", "store": true},
            "title": { "type": "text", "store": true},
            "jd": { "type": "text", "store": true}
        }
    }
}
```

- 查看索引

```shell
# 查看所有索引
GET _cat/indices
# 指定索引查看
GET /job_idx/_mapping
```

- 删除索引

```shell
delete /job_idx
```

- 指定分词器创建索引

```json
PUT /job_idx
{
    "mappings": {
        "properties" : {
            "area": { "type": "text", "store": true, "analyzer": "ik_max_word"},
            "exp": { "type": "text", "store": true, "analyzer": "ik_max_word"},
            "edu": { "type": "keyword", "store": true},
            "salary": { "type": "keyword", "store": true},
            "job_type": { "type": "keyword", "store": true},
            "cmp": { "type": "text", "store": true, "analyzer": "ik_max_word"},
            "pv": { "type": "keyword", "store": true},
            "title": { "type": "text", "store": true, "analyzer": "ik_max_word"},
            "jd": { "type": "text", "store": true, "analyzer": "ik_max_word"}
        }
    }
}
```

- 添加数据

```json
PUT /job_idx/_doc/29097
{
    "area": "深圳-南山区",
    "exp": "1年经验",
    "edu": "大专以上",
    "salary": "6-8千/月",
    "job_type": "实习",
    "cmp": "乐有家",
    "pv": "61.6万人浏览过  / 14人评价  / 113人正在关注",
    "title": "桃园 深大销售实习 岗前培训",
    "jd": "薪酬待遇】 本科薪酬7500起 大专薪酬6800起 以上无业绩要求，同时享有业绩核算比例55%~80% 人均月收入超1.3万 【岗位职责】 1.爱学习，有耐心： 通过公司系统化培训熟悉房地产基本业务及相关法律、金融知识，不功利服务客户，耐心为客户在房产交易中遇到的各类问题； 2.会聆听，会提问： 详细了解客户的核心诉求，精准匹配合适的产品信息，具备和用户良好的沟通能力，有团队协作意识和服务意识； 3.爱琢磨，善思考: 热衷于用户心理研究，善于从用户数据中提炼用户需求，利用个性化、精细化运营手段，提升用户体验。 【岗位要求】 1.18-26周岁，自考大专以上学历； 2.具有良好的亲和力、理解能力、逻辑协调和沟通能力； 3.积极乐观开朗，为人诚实守信，工作积极主动，注重团队合作； 4.愿意服务于高端客户，并且通过与高端客户面对面沟通有意愿提升自己的综合能力； 5.愿意参加公益活动，具有爱心和感恩之心。 【培养路径】 1.上千堂课程;房产知识、营销知识、交易知识、法律法规、客户维护、目标管理、谈判技巧、心理学、经济学; 2.成长陪伴：一对一的师徒辅导 3.线上自主学习平台：乐有家学院，专业团队制作，每周大咖分享 4.储备及管理课堂： 干部训练营、月度/季度管理培训会 【晋升发展】 营销【精英】发展规划：A1置业顾问-A6资深置业专家 营销【管理】发展规划：（入职次月后就可竞聘） 置业顾问-置业经理-店长-营销副总经理-营销副总裁-营销总裁 内部【竞聘】公司职能岗位：如市场、渠道拓展中心、法务部、按揭经理等都是内部竞聘 【联系人】 黄媚主任15017903212（微信同号）"
}
```

- 修改数据

```json
POST /job_idx/_update/29097
{
    "doc": {
        "salary": "15-20千/月"
    }
}
```

- 删除数据

```json
DELETE /job_idx/_doc/29097
```

- 批量导入数据

```shell
# 使用Elasticsearch中自带的bulk接口来进行数据导入
curl -H "Content-Type: application/json" -XPOST "up01:9200/job_idx/_bulk?pretty&refresh" --data-binary "@job_info.json" 
```

- 查看索引状态

```shell
GET _cat/indices?index=job_idx
```

- 根据ID检索数据

```shell
GET /job_idx/_search
{
    "query": {
        "ids": {
            "values": ["46313"]
        }
    }
}
```

- 根据关键词搜索

```json
// 单字段搜索
GET  /job_idx/_search 
{
    "query": {
        "match": {
            "jd": "销售"
        }
    }
}

// 多字段搜索
GET /job_idx/_search
{
    "query" : {
        "multi_match" : {
            "fields" : ["title","jd"],
            "query" : "销售"
        }
    }
}
```

- 分页搜索

```json
GET /job_idx/_search
{
    "from" : 0,
    "size" : 5,
    "query" : {
        "multi_match" : {
            "fields" : ["title", "jd"],
            "query" : "销售"
        }
    }
}
```

## ES操作使用(Curl)

> 最原始的一种方式，搭建完ElasticSearch就能即刻开箱使用，内网环境必备技能。

东西很多，先抓最主要的东西：

- 查看ElasticSearch中的全部索引(查看有多少表)
  - `curl -u elk http://192.168.88.161:9201/_cat/indices?v`
- 查看指定索引的全部field(查看某张表的字段)
  - `curl -XGET -u elk http://192.168.88.161:9201/testindex/_mapping?pretty`
- 根据field进行条件过滤(根据字段进行过滤)
  - `curl -XGET "http://192.168.88.161:9201/testindex/_search" -H 'Content-Type: application/json' -d '
    {
      "query": {
        "bool": {
          "must": [
            { "match": { "mchnt_id": "1" } },
            { "match": { "mchnt_name": "利耶尼亚" } }
          ]
        }
      }
    }'`



```shell
#查看集群状态
curl -u elk http://192.168.88.161:9201/_cat/health?v

#修改集群访问密码
# 在集群的一个节点修改后，会自动同步到其他节点。将 elk 用户的密码更新为 “new_password” 的命令如下。
curl -H "Content-Type: application/json" --user elk -XPUT 'http://192.168.88.161:9201/_xpack/security/user/elk/_password?pretty' -d '{"password":"123456"}'

# 查看集群中的节点状态
curl -u elk http://192.168.88.161:9201

# 查看索引列表
curl -u elk http://192.168.88.161:9201/_cat/indices?v

# 删除索引
# 将索引 testmapping 删除的命令如下。
curl -H "Content-Type: application/json" -u elk -XDELETE http://192.168.88.161:9201/testmapping

# 创建索引
# elkSearch 索引名字须为小写。创建名为 testIndex 的索引会报错
curl -H "Content-Type: application/json" -u elk -XPUT http://192.168.88.161:9201/testindex

# 查看索引结构
# 创建完索引后，查询索引 testindex 包含哪些 field 的命令如下所示。
curl -XGET -u elk http://192.168.88.161:9201/testindex/_mapping?pretty


# 给索引 testindex 设置 2 个 filed，名字分别为 mchnt_id 和 mchnt_name
curl -X PUT 'http://192.168.88.161:9201/testindex/_mapping/doc' -H "Content-Type: application/json" -d '{"properties":{"mchnt_id" : {"type" : "keyword","index" : true},"mchnt_name" : {"type" : "text"}}}'

# 给索引 testindex 新增一个 field，名字为 amount，语句如下所示。
 curl -H "Content-Type: application/json" -u elk -XPOST 'http://192.168.88.161:9201/testindex/_mapping?pretty' -d '{"properties": {"id":{"type":"integer"}}}'


# 添加文档
# 给索引 testindex 添加一条文档，文档 ID 为 1，语句如下。
 curl -H "Content-Type: application/json" -XPOST 'http://192.168.88.161:9201/testindex/doc/2?pretty' -d '{"mchnt_id": "2","mchnt_name": "颐和山庄"}'

# 删除文档
# 将 testindex 索引下，文档 ID 为 3 的文档删除，命令如下。
 curl -u elk -H "Content-Type: application/json" -XDELETE 'http://192.168.88.161:9201/testindex/doc/3?pretty'

# 统计索引 testindex 下的文档数
 curl -s --user elk -XGET 'http://192.168.88.161:9201/_cat/indices/testindex?v' | awk -F ' ' {'print $7'} | grep -v docs.count

# 查看索引 testindex 下的全部数据
 curl -u elk -XGET 'http://192.168.88.161:9201/testindex/_search?pretty=true'

# 搜索 testindex 索引下的全部数据
 curl --user elk -XGET 'http://192.168.88.161:9201/testindex/_search?pretty' -H 'Content-Type:application/json' -d '{"query":{"match_all":{}}}'

# 根据文档 ID 精准获取索引数据
# 精准查询 testindex 索引下的文档 ID 为 20200634 的数据，命令如下。
 curl -H "Content-Type:application/json" -u elk -XGET http://192.168.88.161:9201/testindex/doc/20200634?pretty

# 向某个索引添加field，并指定field的类型
curl -X PUT "http://192.168.88.161:9201/testindex/_mapping/doc" -H 'Content-Type: application/json' -d '{ "properties":{"newslot" : {"type" : "float"}} }'


# 条件查询（如果该fields不支持索引[index]则会查询报错）
curl -XGET "http://192.168.88.161:9201/testindex/_search" -H 'Content-Type: application/json' -d '
{
  "query": {
    "bool": {
      "must": [
        { "match": { "mchnt_id": "1" } },
        { "match": { "mchnt_name": "利耶尼亚" } }
      ]
    }
  }
}'
```



### 注意事项

- 为什么直接写IP？因为没有配置hosts。
- 为什么是9201端口？这是在ElasticSearch安装目录下的config配置文件中配置的，可以自己定义。
- 若集群未添加用户安全认证，“-u elk” 或 “- -user elk” 可以缺省，回车后也无需输入密码。

- 若ElasticSearch 集群添加了用户安全认证功能，curl 命令中的 “-u elk” 代表以 elk 用户访问 elkSearch 集群，也可以修改为 “- -user elk”，命令回车后需要输入密码。
- 如果碰到什么问题，及时参考官方文档，ElasticSearch社区太有活力了！

## ES评分模型

Elasticsearch是基于Lucene的，所以它的评分机制也是基于Lucene的。在Lucene中把这种相关性称为得分（score），确定文档和查询有多大相关性的过程被称为打分（scoring）。

ES最常用的评分模型是 `TF/IDF`和`BM25`，TF-IDF属于向量空间模型，而BM25属于概率模型，但是他们的评分公式差别并不大，都使用IDF方法和TF方法的某种乘积来定义单个词项的权重，然后把和查询匹配的词项的权重相加作为整篇文档的分数。

在ES 5.0版本之前使用了TF/IDF算法实现，而在5.0之后默认使用BM25方法实现。

- Term Frequency(TF)词频：即单词在该文档中出现的次数，词频越高，相关度越高。
- Document Frequency(DF)文档频率：即单词出现的文档数。
- Inverse Document Frequency(IDF)逆向文档频率：与文档频率相反，简单理解为1/DF。即单词出现的文档数越少，相关度越高。
- Field-length Norm：文档越短，相关性越高，field长度，field越长，相关度越弱

TF/IDF模型是Lucene的经典模型，其计算公式如下：

![img](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/13587608-9cfdcee52bbeb848.png)

BM25 模型中BM指的Best Match，25指的是在BM25中的计算公式是第25次迭代优化，是针对TF/IDF的一个优化，其计算公式如下：

![img](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/13587608-26ea90d42b067b7a.png)

- 通过执行计划查看打分过程

```json
GET /job_idx/_search
{
    "explain" : true,
    "from" : 0,
    "size" : 5,
    "query" : {
        "multi_match" : {
            "fields" : ["title", "jd"],
            "query" : "销售"
        }
    }
}
```

## ES编程实现

- 创建虚拟环境

```shell
conda create -n es_env python==3.7.13
并在此环境中安装elasticsearch库:
进入虚拟环境: conda activate es_env
安装python操作ES的库: pip install elasticsearch==7.17.3 -i 清华源
```

- pycharm连接远程环境
- python操作es

```python
class JobClazz():

    def __init__(self, id, area, exp, edu, salary, job_type, cmp, pv, title, jd):
        """
        全参构造函数
        :param id:
        :param area:
        :param exp:
        :param edu:
        :param salary:
        :param job_type:
        :param cmp:
        :param pv:
        :param title:
        :param jd:
        """
        self.id = id
        self.area = area
        self.exp = exp
        self.edu = edu
        self.salary = salary
        self.job_type = job_type
        self.cmp = cmp
        self.pv = pv
        self.title = title
        self.jd = jd


    def getJobClazzDict(self):
        return {
            "id" : self.id,
            "area": self.area,
            "exp": self.exp,
            "edu": self.edu,
            "salary": self.salary,
            "job_type": self.job_type,
            "cmp": self.cmp,
            "pv": self.pv,
            "title": self.title,
            "jd": self.jd
        }
```



```python
from elasticsearch import Elasticsearch

from com.itheima.es.JobClazz import JobClazz

if __name__ == '__main__':
    # 创建连接
    es = Elasticsearch(hosts='192.168.88.166')

    print(es)

    # 添加数据
    document = {"area": "工作地区：郑州-航空港区",
                "exp": "应届毕业生",
                "edu": "博士",
                "salary": "¥ 2千/月",
                "job_type": "全职",
                "cmp": "链家",
                "pv": 963,
                "title": "销售姐姐",
                "jd": "做销售选链家五大理由 一、百分百真房源，放心买房找链家 二、科技化平台支持：链家网、贝壳找房网 三、市场占有率全市前列 四、没有空降兵，所有的管理者都是内部员工晋升 五、我们的理念：客户至上.诚实可信"}
    job_clazz = JobClazz("00001", "工作地区：郑州-航空港区", "应届毕业生", "博士", "¥ 2千/月", "全职", "链家",
                         "9236人浏览过  / 8000人正在关注", "销售姐姐",
                         "做销售选链家五大理由 一、百分百真房源，放心买房找链家 二、科技化平台支持：链家网、贝壳找房网 三、市场占有率全市前列 四、没有空降兵，所有的管理者都是内部员工晋升 五、我们的理念：客户至上.诚实可信")
    print(job_clazz.getJobClazzDict())
    res1 = es.index(index='job_idx', id='00001', document=job_clazz.getJobClazzDict())
    print(res1)

    # 删除数据
    # 不存在的数据会报错
    # res2 = es.delete(index='job_idx', id='46313')
    # print(res2)

    # 修改数据
    res3 = es.update(index='job_idx', id='00001', doc={'salary': '$ 2千/月'})
    print(res3)

    # 根据ID查询
    res4 = es.get(index='job_idx', id='00001')
    print(res4)

    # 根据字段进行搜索
    res5 = es.search(index='job_idx', query={
        "match": {
            "title": "经理"
        }
    }, from_=0, size=5)

    print(res5)

    # 高亮显示
    res6 = es.search(index='job_idx', query={"match": {"title": "经理"}}, highlight={"fields": {"jd": {}, "title": {}}})
    print(res6)

```



- 高亮显示

1、负责公司产品<em>销</em><em>售</em>线索和机会挖掘

> 1、负责公司产品<em>销</em><em>售</em>线索和机会挖掘

## ES读写流程

### 写入ES

![image-20240427174820571](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20240427174820571.png)

- 客户端选择一个DataNode（node2）节点，发起写入请求，此时node2就是一个协调节点
- 协调节点会进行路由（hash(routing) % num of primary shard）routing默认每个document的id
- 对到对应的主分片处理写入请求，将数据写入到index中（将数据保存在_source字段，根据分词构建倒排索引）
- 副本分片从主分片同步数据
- 当主分片和副本分片数据都写入成功将结果返回给客户端

### 检索ES

![image-20230511172503772](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20230511172503772.png)

- 客户端选择一个DataNode（node2）节点，发起查询请求，此时node2就是一个协调节点
- 协调节点将查询请求进行广播
- 每个节点会对每个分片（多个文件，每个文件对应一个倒排索引）根据倒排索引进行查询，将查询的文档ID、分数、分片信息返回给协调节点
- 协调节点汇总（排序）每个数据节点发来的文档信息发送get(根据ID)请求，获取最终结果返回给客户端

### 准实时索引实现

- 溢写到文件系统缓存

![image-20230511173001417](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20230511173001417.png)

当数据写入到ES分片时，会首先写入到内存中，然后通过内存的buffer生成一个segment（每个segment对应一个倒排索引），并刷到文件系统缓存中，数据可以被检索（注意不是直接刷到磁盘）

ES中默认1秒，refresh一次

- 写translog保障容错

![image-20230511173118524](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20230511173118524.png)

在写入到内存中的同时，也会记录translog日志，在refresh期间出现异常，会根据translog来进行数据恢复

等到文件系统缓存中的segment数据都刷到磁盘中，清空translog文件

- flush到磁盘

![image-20230511173133044](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20230511173133044.png)

ES默认每隔30分钟会将文件系统缓存的数据刷入到磁盘

- segment合并

Segment太多时，ES定期会将多个segment合并成为大的segment，减少索引查询时IO开销，此阶段ES会真正的物理删除（之前执行过的delete的数据）

![image-20230802144141483](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20230802144141483.png)

# ES支持SQL

- Elasticsearch SQL特点：

> - 本地集成
> - Elasticsearch SQL是专门为Elasticsearch构建的。每个SQL查询都根据底层存储对相关节点有效执行。
> - 没有额外的要求
> - 不依赖其他的硬件、进程、运行时库，Elasticsearch SQL可以直接运行在Elasticsearch集群上
> - 轻量且高效
> - 像SQL那样简洁、高效地完成查询

- SQL与Elasticsearch对应关系

> | SQL                | Elasticsearch         |
> | ------------------ | --------------------- |
> | column（列）       | field（字段）         |
> | row（行）          | document（文档）      |
> | table（表）        | index（索引）         |
> | schema（模式）     | mapping（映射）       |
> | database（数据库） | Elasticsearch集群实例 |

- Elasticsearch SQL语法

```sql
SELECT select_expr [, ...]
[ FROM table_name ]
[ WHERE condition ]
[ GROUP BY grouping_element [, ...] ]
[ HAVING condition]
[ ORDER BY expression [ ASC | DESC ] [, ...] ]
[ LIMIT [ count ] ]
[ PIVOT ( aggregation_expr FOR column IN ( value [ [ AS ] alias ] [, ...] ) ) ]
```

- 查询数据

```sql
-- 分页查询
GET /_sql?format=txt
{
    "query": "SELECT * FROM job_idx limit 1"
}
-- 将SQL转化为DSL
GET /_sql/translate
{
    "query": "SELECT * FROM job_idx limit 1"
}

-- 匹配查询
GET /_sql?format=txt
{
    "query": "select * from job_idx where MATCH(title, 'hadoop') or MATCH(jd, 'hadoop') limit 10"
}
```

- 案例实现

  - 创建索引

  ```json
  PUT /order_idx/
  {
      "mappings": {
          "properties": {
              "id": {
                  "type": "keyword",
                  "store": true
              },
              "status": {
                  "type": "keyword",
                  "store": true
              },
              "pay_money": {
                  "type": "double",
                  "store": true
              },
              "payway": {
                  "type": "byte",
                  "store": true
              },
              "userid": {
                  "type": "keyword",
                  "store": true
              },
              "operation_date": {
                  "type": "date",
                  "format": "yyyy-MM-dd HH:mm:ss",
                  "store": true
              },
              "category": {
                  "type": "keyword",
                  "store": true
              }
          }
      }
  }	
  ```

  

  - 导入测试数据

  ```shell
  curl -H "Content-Type: application/json" -XPOST "up01:9200/order_idx/_bulk?pretty&refresh" --data-binary "@order_data.json"
  ```

  - 统计不同支付方式的的订单数量

  ```sql
  -- SQL方式
  GET /_sql?format=txt
  {
      "query": "select payway, count(*) as order_cnt from order_idx group by payway"
  }
  
  -- DSL 方式
  GET /order_idx/_search
  {
      "size": 0,
      "aggs": {
          "group_by_state": {
              "terms": {
                  "field": "payway"
              }
          }
      }
  }
  ```

`目前Elasticsearch SQL还存在一些限制。例如：目前from只支持一个表、不支持JOIN、不支持较复杂的子查询。所以，有一些相对复杂一些的功能，还得借助于DSL方式来实现。`

# ES整合HIVE

## 搭建离线数仓

将Mysql中的业务数据导入至Hive：

> 用户表：tbl_users   950
>
> 订单数据表：tbl_orders 120125
>
> 订单商品表：tbl_goods 1
>
> 行为日志表：tbl_logs 376983

## 通过sqoop建表

- 创建用户表 

```shell
# create-hive-table 创建一个Hive表, 读取mysql的表结构, 使用这个结构来创建Hive表
/export/server/sqoop/bin/sqoop create-hive-table \
--connect jdbc:mysql://up01:3306/tags_dat \
--table tbl_users \
--username root \
--password 123456 \
--hive-table tags_dat2.tbl_users \
--fields-terminated-by '\t' \
--lines-terminated-by '\n'
```

- 订单表

```shell
/export/server/sqoop/bin/sqoop create-hive-table \
--connect jdbc:mysql://up01:3306/tags_dat \
--table tbl_orders \
--username root \
--password 123456 \
--hive-table tags_dat2.tbl_orders \
--fields-terminated-by '\t' \
--lines-terminated-by '\n'
```

- 商品表 \001 \x01

```shell
/export/server/sqoop/bin/sqoop create-hive-table \
--connect jdbc:mysql://up01:3306/tags_dat \
--table tbl_goods \
--username root \
--password 123456 \
--hive-table tags_dat2.tbl_goods \
--fields-terminated-by '\t' \
--lines-terminated-by '\n'
```

- 日志表

```shell
/export/server/sqoop/bin/sqoop create-hive-table \
--connect jdbc:mysql://up01:3306/tags_dat \
--table tbl_logs \
--username root \
--password 123456 \
--hive-table tags_dat2.tbl_logs \
--fields-terminated-by '\t' \
--lines-terminated-by '\n'

```

## sqoop导入数据

- 用户数据

```shell
#   direct 直接导出模式 会加快导出速度 使用关系型数据库自带的导出工具(mysql 会使用mysqldump命令)
/export/server/sqoop/bin/sqoop import \
--connect jdbc:mysql://up01:3306/tags_dat \
--username root \
--password 123456 \
--table tbl_users --direct --hive-overwrite --delete-target-dir --fields-terminated-by '\t' --lines-terminated-by '\n' --hive-table tags_dat2.tbl_users --hive-import --num-mappers 1
```

- 订单数据

```shell
/export/server/sqoop/bin/sqoop import \
--connect jdbc:mysql://up01:3306/tags_dat \
--username root \
--password 123456 \
--table tbl_orders --direct --hive-overwrite --delete-target-dir --fields-terminated-by '\t' --lines-terminated-by '\n' --hive-table tags_dat2.tbl_orders --hive-import --num-mappers 10
```

- 商品数据

```shell
/export/server/sqoop/bin/sqoop import \
--connect jdbc:mysql://up01:3306/tags_dat \
--username root \
--password 123456 \
--table tbl_goods --direct --hive-overwrite --delete-target-dir --fields-terminated-by '\t' --lines-terminated-by '\n' --hive-table tags_dat2.tbl_goods --hive-import --num-mappers 5
```

- 行为日志

```shell
/export/server/sqoop/bin/sqoop import \
--connect jdbc:mysql://up01:3306/tags_dat \
--username root \
--password 123456 \
--table tbl_logs --direct --hive-overwrite --delete-target-dir --fields-terminated-by '\t' --lines-terminated-by '\n' --hive-table tags_dat2.tbl_logs --hive-import --num-mappers 20
```

## HIVE写入ES

把Hive数据导入ES 要借助一个工具，es-hadoop 是一个jar 工具也有版本号, 要跟Elasticsearch的版本对应, 当前用的是7.10的ES es-hadoop也要用7.10版本的es-hadoop，使用ES-Hadoop 既可以和Hadoop相关组件交互, 也可以和Spark之间进行交互。

hive数据导入ES步骤

- ① 把es-hadoop的jar包放到hdfs上,当前的虚拟机是放到了/libs/es-hadoop/这个目录下
- ② 在hive的命令行里执行 

```sql
add jar hdfs:///libs/es-hadoop/elasticsearch-hadoop-7.10.2.jar;
# 只对当前的连接生效
```

- ③ 创建一个外部表

```sql
create external table XXX (...)
stored by 'org.elasticsearch.hadoop.hive.EsStorageHandler'  -- 指定了使用eshadoop 中哪个文件来处理数的导入
    tblproperties('es.resource'='tfec_tbl_goods',  -- 数据要导入到哪个ES的索引中
    'es.nodes'='up01:9200',              -- es集群的ip地址
    'es.index.auto.create'='TRUE',     -- 如果es中没有对应的index 自动创建一个
    'es.index.refresh_interval' = '-1',   -- 关掉refresh 导入数据的时候先不创建索引
    'es.index.number_of_replicas' = '0',   -- 设置副本数量 es8才会起作用 当前es7 这个没用
    'es.batch.write.retry.count' = '6',   -- 传输出现问题的时候, 重试的次数
    'es.batch.write.retry.wait' = '60s',  -- 传输等待多久没有响应进行重试 60s没有响应, 就重新连接
    'es.mapping.names' =   'id:id,siteid:siteid' -- hive字段和ES字段之间的映射关系, 如果hive字段和es字段名字一样, 这个不需要设置, 默认es会使用hive的字段名字和数据类型, 如果需要修改映射关系 hive字段1名字:es字段1名字,hive字段2名字:es字段2名字 ...
    );
```

- ④ 查询hive 表的数据,插入到上面创建的外部表中

```sql
insert overwrite table 外部表 select 字段 from hive表;
```

问题：集群告警

解决方案：修改副本数为0

```shell
.es集群告警的问题，可以通过命令解决，官方解决方式，测试后确认无法通过配置文件方式解决：
curl -XPUT http://192.168.88.166:9200/_settings?pretty -d '{ "index": { "number_of_replicas": 0 } }' -H "Content-Type: application/json"
```

问题：数据重复

解决方案：开启upsert模式

> - 'es.write.operation' = 'upsert'
> - 使用upsert模式, 需要指定id
> - 'es.mapping.id' = 'id' 可以选择数据中, 没有重复数据的字段, 做为_id
> - 如果不指定es.mapping.id，es会默认自动生成一个不重复的数据作为_id

问题：索引不刷新

解决方案：修改refresh

```json
PUT ec_tbl_orders/_settings
{
    "index" : {
        "refresh_interval" : "1m"
    }
}

PUT ec_tbl_goods/_settings
{
    "index" : {
        "refresh_interval" : "1m"
    }
}

PUT ec_tbl_logs/_settings
{
    "index" : {
        "refresh_interval" : "1m"
    }
}

PUT ec_tbl_users/_settings
{
    "index" : {
        "refresh_interval" : "1m"
    }
}
```

# ES整合SPARK

`spark 整合 ES , 先把ES-Hadoop的jar包放到 spark的jars目录下`

```shell
/root/anaconda3/envs/pyspark_env/lib/python3.7/site-packages/pyspark/jars
```

- Spark读取ES代码

```python
spark.read.format('es').option()
```

- 必选参数

```properties
es.resource 读取的index名称
es.nodes es集群的连接地址
```

- 可选参数

```properties
es.index.auto.create (default yes)
Whether elasticsearch-hadoop should create an index (if its missing) when writing data to Elasticsearch or fail.当将数据到es中的时候是否会自动创建索引，默认会自动创建索引或索引库
---------------------------------------------------------------
es.index.read.missing.as.empty (default no)
Whether elasticsearch-hadoop will allow reading of non existing indices (and return an empty data set) or not (and throw an exception)是否可以读取空索引，默认是no不读取
---------------------------------------------------------------
es.query (default none)
Holds the query used for reading data from the specified es.resource. By default it is not set/empty, meaning the entire data under the specified index/type is returned. es.query can have three forms:
保存用于从指定es读取数据的查询。资源默认情况下，它不设置/为空，这意味着返回指定索引/类型下的整个数据。查询可以
uri query 作为上述的参数
using the form ?uri_query, one can specify a query string. Notice the leading ?.
----------------------------------------------------------------------
es.write.operation (default index)
The write operation elasticsearch-hadoop should perform - can be any of:

（1）index (default)
new data is added while existing data (based on its id) is replaced (reindexed).
添加新数据的同时替换（重新索引）现有数据（基于其id）。
（2）create
添加新数据-如果数据已经存在（基于其id），将引发异常。
adds new data - if the data already exists (based on its id), an exception is thrown.
（3）update
updates existing data (based on its id). If no data is found, an exception is thrown.
更新现有数据（基于其id）。如果找不到数据，将引发异常。
（4）upsert
known as merge or insert if the data does not exist, updates if the data exists (based on its id).
如果数据不存在，则称为合并或插入；如果数据存在，则更新（基于其id）。
（5）delete
deletes existing data (based on its id). If no data is found, an exception is thrown.
删除现有数据（基于其id）。如果找不到数据，将引发异常。
```

# 注意

- python操作es
  - from elasticsearch import Elasticsearch  (pip安装对应版本依赖)
  - es = Elasticsearch(hosts=‘up01:9200’)

- sql操作es
  - 本地原生支持
  - 目前不支持join操作
- es读写流程
  - 写入数据
    - 客户端发送写入请求协调节点（随机选择）
    - 协调节点根据数据的routing(_id)计算hash值，并根据主分片的数量进行路由
    - 找到对应主分片将数据写入保存并创建倒排索引
    - 副本分片从主分片上同步数据
    - 当数据保存完毕后由协调节点向客户端返回结果
  - 读取数据
    - 客户端发送读取数据的请求给协调节点
    - 协调节点对读取请求进行广播
    - 所有分片都会进行数据查询
    - 将查询结果（文档ID、分片信息、分数等）在协调节点上进行汇总（全局排序）
    - 按照汇总的结果向对应的分片发送get请求获取数据并返回给客户端
  - 索引实现
    - 数据首先写入内存的buffer中，同时会写一份数据到磁盘上translog中，用来保证数据的安全
    - 默认1秒进行一次refresh，将内存buffer中的数据写入到文件缓冲区生成一个segment，同时生成对应的倒排索引
    - 默认30分钟进行一次flush，将文件缓冲区中的segment刷新到磁盘上进行持久化保存，同时清空内存和translog中持久化后的数据 
    - 由于segment会生成很多小文件，会定期进行小segment合并成大segment，同时会将之前delete过的数据做真正物理上的删除
- es整合hive
  - 上传elasticsearch-hadoop.xx对应的版本jar包至hdfs
  - 在当前hive会话 add jar jar的路径（只在当前会话生效）
  - 使用hive创建一个映射es的外部表，create external table 表名(schema) sorted by “xxxx”  tblproperties(key=value)
  - hive->es ： insert overwrite 外部表 from hive表
- es整合spark
  - 需要将elasticsearch-hadoop.xxx.jar放到pyspark运行环境下
  - sparkSession.read.format(‘es’).option(‘es.nodes’, xxx).option(‘es.resource’, xxxx).option(‘es.read.field.include’, xxxx)
  - sparkSession.write.format(‘es’)









