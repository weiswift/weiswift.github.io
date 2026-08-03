---
title: Kafka（全）
date: 2024-04-24 00:00:00
author: JohnsonLiam
tags: [Kafka]
top_img: https://wei-blog.oss-cn-beijing.aliyuncs.com/img/ChMkKmAwfdGIV1LgABJv3n4v8X4AAKLSgCDj14AEm_2352.jpg
comments: true
---

# :happy:消息队列

## 什么是消息队列

![image-20221021095506979](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20221021095506979.png)

- 队列：是一种先进先出的数据结构
- 消息队列：是由生产者将数据从一端放入消息队列，由消费者按顺序从另一端进行取出使用

## 消息队列的应用场景

> 消息队列在实际应用中包括如下四个场景：
>
> 1) 应用解耦：多应用间通过消息队列对同一消息进行处理，避免调用接口失败导致整个过程失败；
> 2) 异步处理：多应用对消息队列中同一消息进行处理，应用间并发处理消息，相比串行处理，减少处理时间；
> 3) 限流削峰：广泛应用于秒杀或抢购活动中，避免流量过大导致应用系统挂掉的情况；
> 4) 消息驱动的系统：系统分为消息队列、消息生产者、消息消费者，生产者负责产生消息，消费者(可能有多个)负责对消息进行处理

## 消息队列的两种模式

### 点对点模式

> - 每个消息只能被一个消费者消费，一个生产者相当于对应一个消费者
> - 消息被消费之后就删除
> - 一对一

### 发布订阅模式

> - 每个消息可以被多个消费者消费
> - 消息被消费后不会被删除
> - 一对多

## 常见的消息队列产品



<img src="https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20220707102731798.png" alt="image-20220707102731798" style="zoom:50%;" />

# :happy:Kafka的简介

## Kafka的基本介绍

> Apache Kafka is an open-source distributed event streaming platform used by thousands of companies for high-performance data pipelines, streaming analytics, data integration, and mission-critical applications.
>
> - 开源-免费、使用广
> - 分布式-可横向扩展
> - 事件流和数据管道-实时消息处理
> - 流数据分析
> - 数据整合
> - 去中心化（没有master）

> 特点：
>
> - 并发性：分布式, 分区 
> - 可靠性： 副本和容错等
> - 可扩展性： kakfa消息传递系统轻松缩放, 无需停机
> - 耐用性： kafka使用分布式提交日志, 这个意味着消息会尽可能快速的保存在磁盘上, 因此它是持久的
> - 性能： kafka对于发布和订阅消息都具有高吞吐量, 即使存储了许多TB的消息, 他也爆出稳定的性能- kafka非常快: 保证零停机和零数据丢失

## Kafka的使用场景

> - 基本上消息队列的使用场景都能满足
> - 在日志采集方面使用比较多（配合flume）
> - 大数据领域的消息总线

# :kissing:Kafka的基本架构

## Kafka的基础构架

![image-20221021112403987](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20221021112403987.png)

> - 生产者-producer：负责生产消息（谁往Kafka中生产消息谁就是生产者）
> - 消费者-consumer：负责消费消息（谁从Kafka中消费消息谁就是消费者）
> - 运行实例-broker： Kafka实际工作的进程
> - 主题-topic：一类消息的集合，消息往哪放从哪取相当于数据库中的表
> - 分区-partition：数据的分区
> - 副本-replica：数据的副本
> - 主副本-leader replica：实际负责数据读写的副本，生产者和消费者都与这个副本进行交互的
> - 从副本-follower replica：负责从主副本上同步数据，实现数据备份，保证数据可靠性
> - 消费者组-consumer group：多个消费者的集合
> - AR：ALL REPLICA所有副本的集合，等于ISR+OSR
> - ISR： In Sync Replica数据同步成功的副本（实际可用的副本）
> - OSR： Out of Sync Replica数据同步不成功的副本（不可用的副本）

# :grin:Kafka的集群搭建

> 参考：安装部署文档
>
> `kafka集群需要zookeeper集群（Kafka3.0可以独立运行不再需要zookeeper）`

# :grin:Kafka的shell命令

## 启动集群

```shell
# 三个节点分别执行
nohup bin/kafka-server-start.sh config/server.properties 2>&1 >> kafka.out &
-nohup 守护进程
-& 后台运行
-bin/kafka-server-start.sh 服务启动脚本
-config/server.properties 配置文件
-2>&1 2-系统错误输出 1-系统标准输出 >&重定向
->> kafka.out 追加到kafka.out这个文件中
```

## 创建Topic

```shell
# 创建1副本3分区
bin/kafka-topics.sh --create --zookeeper node1:2181 --replication-factor 1 --partitions 3 --topic test
-bin/kafka-topics.sh 有关topic相关命令脚本
--create 创建动作
--zookeeper node1:2181
--replication-factor 指定副本数量
--partitions 指定分区数量
--topic test 指定topic的名字
```

![image-20221021145836644](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20221021145836644.png)

```shell
# 创建3副本3分区
bin/kafka-topics.sh --create --zookeeper node1:2181 --replication-factor 3 --partitions 3 --topic test_3r
```

`注意：已经创建过的topic不可重复创建`

![image-20221021145226354](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20221021145226354.png)

## 模拟生产消费

```shell
# 启动一个模拟生产者
bin/kafka-console-producer.sh --broker-list node1:9092 --topic test 
# 启动一个模拟消费者
bin/kafka-console-consumer.sh --bootstrap-server node1:9092 --topic test --from-beginning
--from-beginnig 从最初开始消费
```

## 查看Topic列表

```shell
bin/kafka-topics.sh --list --zookeeper node1:2181
```

## 查看Topic详情

```shell
bin/kafka-topics.sh --describe --zookeeper node1:2181 --topic test
```

## 删除Topic

```shell
bin/kafka-topics.sh --delete --zookeeper node1:2181 --topic test
```

## 修改Topic

```shell
# 修改分区数量
bin/kafka-topics.sh --alter --zookeeper node1:2181 --topic test --partitions 2
```

![image-20221021144728135](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20221021144728135.png)

`注意：topic的副本是不可修改的`

![image-20221021145042654](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20221021145042654.png)

# :happy:Kafka tools

> 安装程序在提供的前期资料里

# :grin:Kafka基准测试

## 1分区1副本

- ### 生产者能力测试

```shell
# 创建测试用的topic
bin/kafka-topics.sh --zookeeper node1.itcast.cn:2181 --create --topic benchmark --partitions 1 --replication-factor 1

# 使用测试脚本
bin/kafka-producer-perf-test.sh --topic benchmark --num-records 5000000 --throughput -1 --record-size 1000 --producer-props bootstrap.servers=node1:9092,node2:9092,node3:9092 acks=1
-bin/kafka-producer-perf-test.sh 测试脚本
--topic benchmark 指定要测试的topic
--num-records 5000000 生产的数据量
--throughput -1 限流（-1表示不限）
--record-size 1000 消息的大小，单位bytes
--producer-props 指定broker的地址
-acks=1 应答模式
```

![image-20221021160221807](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20221021160221807.png)

- ### 消费者能力测试

```shell
bin/kafka-consumer-perf-test.sh --broker-list node1:9092,node2:9092,node3:9092 --topic benchmark --fetch-size 1048576 --messages 5000000
--fetch-size 1048576 每次消费的数据大小（1MB）
--messages 5000000 总共消费的数量
```

![image-20221021161027976](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20221021161027976.png)

## 3分区1副本

- #### 生产者

```shell
# 创建测试用的topic
bin/kafka-topics.sh --zookeeper node1.itcast.cn:2181 --create --topic benchmark_3p --partitions 3 --replication-factor 1

# 使用测试脚本
bin/kafka-producer-perf-test.sh --topic benchmark_3p --num-records 5000000 --throughput -1 --record-size 1000 --producer-props bootstrap.servers=node1:9092,node2:9092,node3:9092 acks=1
-bin/kafka-producer-perf-test.sh 测试脚本
--topic benchmark 指定要测试的topic
--num-records 5000000 生产的数据量
--throughput -1 限流（-1表示不限）
--record-size 1000 消息的大小，单位bytes
--producer-props 指定broker的地址
-acks=1 应答模式
```

![image-20221021161706181](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20221021161706181.png)

- #### 消费者

```shell
bin/kafka-consumer-perf-test.sh --broker-list node1:9092,node2:9092,node3:9092 --topic benchmark_3p --fetch-size 1048576 --messages 5000000
--fetch-size 1048576 每次消费的数据大小（1MB）
--messages 5000000 总共消费的数量
```

![image-20221021161921777](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20221021161921777.png)

`注意：这个主题是有三个分区，但是我们只启动了一个消费者，只能三个分区依次消费（一个时刻只能消费分区）`

# :grin:kafka-python API使用

## 安装库

需要安装`kafka-python`

```shell
pip install kafka-python -i https://pypi.tuna.tsinghua.edu.cn/simple
```

## 生产者代码

```python
# coding:utf8
"""
python使用Kafka API
1、获取生产者连接对象
2、向topic发送消息
3、获取反馈信息
"""
from kafka import KafkaProducer

# 获取生产者对象，参数bootstrap_servers broker地址列表，传入list对象，内容是list['host:port', 'host:port', ......]
    """
    常用其它参数：
    - client_id(str) 客户端字符串ID, Default: ‘kafka-python-producer-#’ (appended with a unique number per instance)
    - acks(0, 1, 'all')
    - compression_type(str) 压缩方式，可选：gzip, lz4, snappy, None 默认None
    - retries(int) 重试次数， 默认0
    - batch_size(数字) 批发送大小，默认：16384, 设置为0禁用批处理
    - value_serializer(函数) value的序列化逻辑
    注意：生产者为 异步发送
    """
producer = KafkaProducer(
    bootstrap_servers=['node1:9092', 'node2:9092', 'node3:9092']
)

# 向topic发送数据
for i in range(0, 10):
    furture = producer.send(topic='test', value=f"yangkunlin is handsome{i}".encode("utf-8"))

    # 获取反馈信息
    record_metadata = furture.get(timeout=10)  # 如果超过10s没有反馈就会报错

    # record_metadata中存放了反馈的相关信息
    print(f"发送的topic是：{record_metadata.topic}")
    print(f"发送的分区是：{record_metadata.partition}")

# 提交缓冲区
producer.flush()

```

## 消费者代码

```python
# coding:utf8
"""
python使用Kafka消费者API
1、获取Kafka消费者对象
2、向topic拉取数据进行消费
3、提交消费偏移量
"""
from kafka import KafkaConsumer

# 获取消费者对象
"""
consumer = KafkaConsumer(
        'test',                                     # topic
        group_id='my-group',                        # 消费者组
        bootstrap_servers=['single-bigdata:9092'],  # broker list
        auto_offset_reset='earliest',               # 从头开始消费，可选earliest和latest, 默认latest
        enable_auto_commit=False,                  # 自动提交offset关闭
        auto_commit_interval_ms=5000,   # 自动偏移提交之间的毫秒数，如果 enable_auto_commit 为 True
        client_id="test_consumer",
        key_deserializer=None,
        value_deserializer=None,
        fetch_min_bytes=1,   # 服务器应为获取请求返回的最小数据量，否则等待 fetch_max_wait_ms 以累积更多数据。默认1(byte)
        fetch_max_wait_ms=500,  # 如果没有足够的数据立即满足 fetch_min_bytes 给出的要求，服务器将在响应 fetch 请求之前阻塞的最长时间（以毫秒为单位）
        fetch_max_bytes=52428800 # Default: 52428800 (50 MB). 最大抓取大小达到后立刻返回
        # 以上是常用参数， 详细参见： https://kafka-python.readthedocs.io/en/master/apidoc/KafkaConsumer.html
    )

"""
consumer = KafkaConsumer(
    "test",  # 待消费的主题名
    group_id='mygroup',  # 消费者组，后面说
    bootstrap_servers=['node1:9092', 'node2:9092', 'node3:9092'],  # 服务器地址、端口
    auto_offset_reset='earliest',  # 表示消费者启动的时候从可用的最早数据开始读取
    enable_auto_commit=False,  # 表示手动提交消费的偏移量，如果设置为Ture，默认是1分钟提交一次
    client_id='client1'
)

# 从topic拉取数据消费
for message in consumer:
    # 这个for循环不会停止，会一直监控topic
    # 有数据来就拉取进行消费
    topic = message.topic
    partition = message.partition
    value = message.value
    offset = message.offset

    print("消费的主题是：%s，消费的分区是：%d，消费的值是：%s，消费的偏移量是：%d" % (topic, partition, value, offset))
    # 提交消费的偏移量
    consumer.commit()

```

> - Kafka不能保证消费者消费数据整体有序
> - 只能保证单个分区内的数据有序
>
> ![image-20221021172705468](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20221021172705468.png)

# :kissing:Kafka高级

## Kafka的数据位移

![image-20221022093213291](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20221022093213291.png)

> - 偏移量-生产者在把数据放进消息队列时，每个数据会附带一个offset
> - 位移-消费者在从消息队列中消费数据时，会记录当前消费到什么位置offset
> - 对于消费者消费的位移是保存在Kafka内置的topic：__consumer_offset中的

Kafka数据被消费之后是不会自动删除的

> 达到保存时间，默认是7天，参数在server.properties配置文件中
>
> log.retention.hours=168（单位小时）

不同的消费者可以同时消费同一个主题，但是同一个消费者会不会重复消费呢？

> - 生产者在生产数据的时候会生成对应的offset，也就是当前数据的offset
> - 消费者在消费数据的时候会记录当前已消费数据的offset,也就是当前数据的offset

## Kafka的分片与副本机制

- 分片：对于分布式的系统，可以将大规模的数据分开存储，比如hdfs上会把数据分成不同的block分别存储在不同的datanode上，即提高了存储能力又降低了复杂度，同时可以提高数据处理的并发能力
- 副本：对于分布式的系统，数据分散保存出现风险的机率高，有一个节点出现问题，数据就不完整了，所以可以利用副本的机制提高容错

![image-20221022095902605](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20221022095902605.png)

> - 在Kafka中一个主题可以有多个分区，分区的数量建议是broker数量的N倍，可以更好的利用硬件资源，提高并行效率
> - 一个分区可以设置多个副本，建议副本数不超过3个，即可以满足数据的容错，又不会太过影响性能
> - 副本在broker数量满足的情况下会尽量分布在不同的broker上
> - 副本之间会通过内部机制选举一个Leader副本，剩下的是follow副本，数据会首先写入Leader副本，其它follow副本会自动从Leader上同步数据
> - 一般情况下Kafka集群也就是3、5、7台就够了，可以搭建多个Kafka集群

数据写入成功的标志应该是什么？是写入Leader还是Follow同步数据完成？

## 如何保证数据不丢失

### 生产者端

数据写入：

- 数据成功写入leader
- follow成功从leader同步数据

数据在写入leader之后有三种选择，取决于ack的值：

- 当ack=0时，生产者将数据写入leader后，就不管了，此时直接认定写入成功，写性能最高

- 当ack=1时，生产者将数据写入leader后，等待leader反馈，leader告诉生产者写入成功了，生产者才离开认定ok

- 当ack=-1时，生产者将数据写入leader后，会多等待一会，等leader反馈成功加上follow同步数据成功才离开认定ok

安全性：

- ack=0几乎没有安全性，但是写入性能是最好的
- ack=1有一定的安全性，能够确保数据成功写入leader，但是不确保副本同步数据成功，如果leader发生故障会造成数据丢失
- ack=-1安全性棒棒的，但是写入性能相对较低

> 在Kafka中：
>
> - 所有副本统称为AR
> - 同步数据正常的副本统称为ISR : In-Sync Replicas副本同步队列
> - 同步数据不正常的副本叫OSR ：Out-Of-Sync Replicas

写入的模式：

- 同步：顺序执行，写入一条等待反馈，再写下一条
- 异步：先将数据写入缓冲区，批量写入Kafka等待反馈

数据复制的过程，通过HW机制保证消费数据可靠性（只考虑ISR队列）：

![image-20221022103507299](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20221022103507299.png)

### 消费者端

消费靠提交offset，记录消费数据的位置

- 如果消费数据成功后再提交offset，可能会重复消费数据
- 如果先提交offset再消费数据，可能会少消费数据

## Kafka的数据存储

### 存储位置

```shell
# 在config/server.properties内
# A comma separated list of directories under which to store log files
log.dirs=/export/data/kafka-data/kafka-logs
```

![image-20221022104733373](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20221022104733373.png)

![image-20221022105138930](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20221022105138930.png)

`Kafka是怎样实现高吞吐的？`

> Kafka官方给出的性能指标能达到500MB/S（写入硬盘的指标），Kafka在写入数据时会预先找一块连续的磁盘空间顺序写入

> - 根据log的文件名称就能快速定位某个offset数据在哪个文件
> - Kafka默认，每写满1GB的.log文件后，就切换下一个文件

### 消费的流程

`稀疏索引`

![image-20221022111645992](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20221022111645992.png)

> 需求：找到offset=1060299这个数据
>
> 1、定位索引文件00000000000001060118.index
>
> 2、通过二分查找快速定位到对应索引区间，比如向上最近的位置：1060200
>
> 3、根据这个最近的索引位置到数据空间中向下搜索
>
> 4、顺序搜索到想要的数据offset=1060299

## 生产者的分发策略

### 内置的分区器

> 分区器负责决定当数据来时，这个数据被分发至哪个分区
>
> ![image-20221022112242056](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20221022112242056.png)

### 指定分区发送

> 通过send方法指定分区转发

```python
furture = producer.send(
        topic='test',
        value=f"yangkunlin is handsome{i}".encode("utf-8"),
        partition=1  # 指定分区发送
    )
```

> `练习`：现在我们有三个分区，需要三个分区转发数据的比例为2:3:5

```python
# 指定三个分区的比例为2:3:5
partitios = [0, 0, 1, 1, 1, 2, 2, 2, 2, 2]

# 向topic生产数据
for i in range(0, 1000):
    future = producer.send(topic='test', \
                           value=f"yangkunlin is handsome{i}" \
                           .encode("utf-8"), \
                           partition=partitios[random.randint(0, 9)])
```



### 指定key分发数据

```python
furture = producer.send(
        topic='test',
        value=f"yangkunlin is handsome{i}".encode("utf-8"),
        key="yangkunlin"
    )
```

`指定分区的优先级大于指定key`

## 消费者的负载均衡

### 消费者组与分区

- #### 查看当前消费者组列表

```shell
bin/kafka-consumer-groups.sh --bootstrap-server node1:9092,node2:9092,node3:9092 --list
```

- #### 查看消费者组详情信息

```shell
bin/kafka-consumer-groups.sh --bootstrap-server node1:9092,node2:9092,node3:9092 --group mygroup --describe
```

#### 场景一：三个分区，一个消费者组里有一个消费者

![image-20221022113234477](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20221022113234477.png)

> 所有的分区都由这个一个消费进行消费
>
> ![image-20221022113440902](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20221022113440902.png)

#### 场景二：三个分区，一个消费者组里有四个消费者

![image-20221022113625093](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20221022113625093.png)

> - `规则：同一个分区只能分配给一个消费者组内的一个消费者消费`
> - 划分分区时最好保证消费者的数量与分区相等
> - 当消费者数据大于分区数量时，肯定有消费者空闲
>
> ![image-20221022114129835](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20221022114129835.png)

#### 场景三：三个分区，两个消费者组分别有四个消费者

![image-20221022114428214](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20221022114428214.png)

### 数据的堆积

![image-20221022114832021](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20221022114832021.png)

> `LAG='LOG-END-OFFSET' - 'CURRENT-OFFSET'`

# :happy:kafka-eagle安装使用

> 查看安装文档