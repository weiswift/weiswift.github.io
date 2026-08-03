---
title: DolphinScheduler（全）
date: 2024-04-24 00:00:00
author: Johns onLiam
tags: [DS]
top_img: https://wei-blog.oss-cn-beijing.aliyuncs.com/img/224634-171232839454f0.jpg
comments: true
---

# DolphinScheduler任务调度器

## DS基本介绍

> DolphinScheduler是apache旗下一款顶级的工作流调度系统, 早期是由国内易观公司开发, 在2019年贡献给apache, 并成为apache旗下顶级项目, 主要作用: 实现工作流的调度操作 与 oozie是同类型的软件, 只不过比ooize提供了更加友好的操作界面, 可以直接通过界面对工作流进行完整的配置 启动 监控等相关的工作

![image-20220223145625211](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20220223145625211.png)

## 安装DS

* 1- 将提供的DS的安装包拷贝到项目环境的_04_software 目录下

![image-20220110155816749](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20220110155816749.png)

* 2- 将安装包拖拽到node1的 /export/software下

![image-20220110155924941](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20220110155924941.png)

* 3- 进行解压操作, 并配置软连接

```properties
cd /export/software
tar -zxf apache-dolphinscheduler-incubating-1.3.5-dolphinscheduler-bin.tar.gz -C /export/server/

cd /export/server/
ln -s apache-dolphinscheduler-incubating-1.3.5-dolphinscheduler-bin/ dolphinscheduler
```

* 4- 添加mysql的驱动包到 DS的lib目录下

![image-20220110160241674](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20220110160241674.png)

* 5- 修改DS的初始数据源的配置文件

![image-20220110160345163](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20220110160345163.png)

```properties
修改一下内容:
# 此部分添加 # 注释
# postgresql
#spring.datasource.driver-class-name=org.postgresql.Driver
#spring.datasource.url=jdbc:postgresql://localhost:5432/dolphinscheduler
#spring.datasource.username=test
#spring.datasource.password=test

# 新增的内容
# mysql
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
spring.datasource.url=jdbc:mysql://192.168.88.161:3306/dolphinscheduler?characterEncoding=UTF-8&allowMultiQueries=true
spring.datasource.username=root
spring.datasource.password=123456


说明: 
	请注意, 在复制的时候能不能不把中文复制进去? 
```

![image-20220110160620818](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20220110160620818.png)

* 6- 进入mysql的客户端, 执行以下代码:

```properties
CREATE DATABASE dolphinscheduler DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
```

![image-20220110160809818](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20220110160809818.png)

* 7- 初始化元数据表:

```
cd /export/server/dolphinscheduler
sh script/create-dolphinscheduler.sh
```

![image-20220110160949630](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20220110160949630.png)

![image-20220110161019630](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20220110161019630.png)

* 8- 修改 conf/env/dolphinscheduler_env.sh 环境变量

![image-20220110161219928](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20220110161219928.png)

````properties
export HADOOP_HOME=/export/server/hadoop
export HADOOP_CONF_DIR=/export/server/hadoop/etc/hadoop
export SPARK_HOME1=/export/server/spark
#export SPARK_HOME2=/opt/soft/spark2
export PYTHON_HOME=/root/anaconda3/bin/python
export JAVA_HOME=/export/server/jdk1.8.0_241
export HIVE_HOME=/export/server/hive
#export FLINK_HOME=/opt/soft/flink
#export DATAX_HOME=/opt/soft/datax/bin/datax.py
export SQOOP_HOME=/export/server/sqoop

export PATH=$HADOOP_HOME/bin:$HADOOP_CONF_DIR:$PYTHON_HOME:$SPARK_HOME1/bin:$JAVA_HOME/bin:$HIVE_HOME/bin:$SQOOP_HOME/bin:$PATH
````

![image-20220110161831663](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20220110161831663.png)

* 9- 修改 conf/config/install_config.conf (安装配置文件)
  * 说明: 目前DS还没有安装, 仅仅是在配置DS的安装配置文件

![image-20220110162024990](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20220110162024990.png)

```properties
说明: 将配置文件对应配置信息, 改为一下的内容, 与以下内容一定一定要保持一致

dbhost="192.168.88.161:3306"
username="root"
password="123456"
zkQuorum="192.168.88.161:2181,192.168.88.162:2181,192.168.88.163:2181"
installPath="/export/server/dolphinscheduler_install"
deployUser="root"
#mailServerHost="smtp.exmail.qq.com"
#mailServerPort="25"
#mailSender="xxxxxxxxxx"
#mailUser="xxxxxxxxxx"
#mailPassword="xxxxxxxxxx"
#starttlsEnable="true"
#sslEnable="false"
#sslTrust="smtp.exmail.qq.com"
resourceStorageType="HDFS"
defaultFS="hdfs://192.168.88.161:8020"
#yarnHaIps="192.168.xx.xx,192.168.xx.xx"
singleYarnIp="192.168.88.161"
#hdfsRootUser="hdfs"
ips="192.168.88.161,192.168.88.162,192.168.88.163"
masters="192.168.88.161,192.168.88.162"
workers="192.168.88.161,192.168.88.162,192.168.88.163"
alertServer="192.168.88.163"
apiServers="192.168.88.161"
```

![image-20220110163050217](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20220110163050217.png)



-----

* 10 - 启动 zookeeper集群:

```properties
注意: 三个节点都要执行
cd /export/server/zookeeper/bin/
./zkServer.sh start


三个节点启动后, 需要查看zk的状态:
./zkServer.sh status

必须看到: 两个follower 和 一个 leader
```

* 11- 触发安装并启动

```properties
cd /export/server/dolphinscheduler
sh install.sh


注意:
	此操作, 会进行DS的安装操作, 安装完成后, 自动将DS进行启动  
	此操作, 仅需要第一次执行一次即可, 后续启动DS会有专门的命令的
```

安装后, 需要查看各个节点:

node1:

![image-20220110165416910](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20220110165416910.png)

node2:

![image-20220110165442727](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20220110165442727.png)

node3:

![image-20220110165505333](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20220110165505333.png)



----

后续的启动, 是专门有命令来处理的:

```shell
cd /export/server/dolphinscheduler_install

一键停止集群所有服务
sh ./bin/stop-all.sh
一键开启集群所有服务
sh ./bin/start-all.sh

单独停止和启动命令:
sh ./bin/dolphinscheduler-daemon.sh start master-server
sh ./bin/dolphinscheduler-daemon.sh stop master-server

sh ./bin/dolphinscheduler-daemon.sh start worker-server
sh ./bin/dolphinscheduler-daemon.sh stop worker-server

sh ./bin/dolphinscheduler-daemon.sh start api-server
sh ./bin/dolphinscheduler-daemon.sh stop api-server

sh ./bin/dolphinscheduler-daemon.sh start logger-server
sh ./bin/dolphinscheduler-daemon.sh stop logger-server

sh ./bin/dolphinscheduler-daemon.sh start alert-server
sh ./bin/dolphinscheduler-daemon.sh stop alert-server
```



访问DS: http://192.168.88.161:12345/dolphinscheduler

用户名: admin

密码: dolphinscheduler123

![image-20220110170047938](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20220110170047938.png)

## DS架构说明

![image-20220223160217067](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20220223160217067.png)

```properties
说明:
	首先由用户在web UI界面上配置工作流, 配置完成后, 启动工作流, 启动后, 启动工作流请求就会发生给 API服务, API服务接收到请求, 会随机的选择一台Master节点 负责这个任务的处理, Master收到任务后, 开始形成DAG执行流程图, 进行任务分配, 确定任务应该交给那个或者那些worker节点来负责, 分配完成后, 将任务信息发送给对应worker节点, 由worker节点来负责最终任务的执行
	同时告警服务也在时刻监控着工作流的执行状态, 一旦发现工作流存在问题, 就会实施告警工作, 同时worker节点还附带一个logger日志服务, 此服务也会实时记录这个各个节点的工作日志, 方面用户进行实时查看执行日志
```



注意: 

```properties
	目前配置DS的高可用集群方案的状态, 当用户提交一个工作流后, 这个工作流最终被那个或那些worker执行是不确定的, 所以后续在执行shell脚本的时候, 由于shell脚本存储在本地的, 需要将shell脚本分发给 node2和node3, 以及sqoop也需要在node2和node3都要安装好
```



## DS的基本使用

### 队列

后续在提交工作流的时候, 可以将工作流提交到指定的队列中, 后续基于队列进行资源化管理

![image-20220110171930190](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20220110171930190.png)

### 租户

建议配置为root, 避免后续有一些权限问题

![image-20220110172147382](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20220110172147382.png)



### 登录用户

此部分可以创建全新的用户, 可以使用这个用户登录DS, 进行相关的操作

![image-20220110172539556](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20220110172539556.png)

### 告警组

说明: 可以配置告警信息, 后续在使用工作流的时候, 就可以配置告警组, 配置后, 对应告警组的用户就会受到告警邮件或者短信

![image-20220110172740509](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20220110172740509.png)

### worker分组说明

说明:

```properties
	此组主要是用于后续在进行工作流执行的时候, 可以指定worker分组, 这样master在进行任务分配的时候,会从对应worker分组中选择一个worker节点来运行, 在实际生产环境中, 此分组可能会很多个, 根据任务的大小来选择对应的分组执行
```

![image-20220110173009145](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20220110173009145.png)

### 创建项目编写工作流

![image-20220110173228377](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20220110173228377.png)

![image-20220110173244500](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20220110173244500.png)

![image-20220110173350247](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20220110173350247.png)

![image-20220110173625540](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20220110173625540.png)

![image-20220110173744090](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20220110173744090.png)

![image-20220110173908617](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20220110173908617.png)

```
说明:
	如果要修改工作流, 必须先将工作流下线, 否则不允许修改
```

![image-20220110174004239](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20220110174004239.png)

执行完成后: 

![image-20220110174205608](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20220110174205608.png)

## 基于DS实现ODS任务工作流配置

* 1- 将 node1中10个采集的shell脚本拷贝到node2和node3: 统一放置位置  /export/data/insurance_collect_sh

```shell
node1: 
	mkdir -p /export/data/insurance_collect_sh
	
	cd /export/data/workspace/itcast_insurance/_02_sh_sqoop/
	cp _* /export/data/insurance_collect_sh/


将数据发送给node2和node3:
	cd /export/data/
	scp -r insurance_collect_sh/ node2:$PWD
	scp -r insurance_collect_sh/ node3:$PWD
	
注意:
	拷贝完成后 记得到node2和node3校验一下
```

* 2- 将sqoop在node2和node3也进行一下安装操作:

```shell
node1: 
	cd /export/server/
	scp -r sqoop-1.4.7.bin__hadoop-2.6.0/ node2:$PWD
	scp -r sqoop-1.4.7.bin__hadoop-2.6.0/ node3:$PWD

在node2和node3中配置软连接以及配置环境变量
	cd /export/server/
	ln -s sqoop-1.4.7.bin__hadoop-2.6.0/ sqoop
	
	vim /etc/profile
	添加以下内容:
	#SQOOP_HOME
    export SQOOP_HOME=/export/server/sqoop
    export PATH=$PATH:$SQOOP_HOME/bin
    
    记得: source /etc/profile
```

* 3- 在node2和node3中安装 dos2unix

```
yum -y install dos2unix
```

* 4- 在DS上进行工作流的配置操作
  * 脚本设置内容如下:

```
dos2unix /export/data/insurance_collect_sh/_01_insurance_mort_10_13_import.sh
sh /export/data/insurance_collect_sh/_01_insurance_mort_10_13_import.sh
```

![image-20220111094827865](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20220111094827865.png)

![image-20220111094926477](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20220111094926477.png)

* 4- 进行工作流的上线操作, 并立即执行

![image-20220111095037534](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20220111095037534.png)



说明:

```properties
	建议在运行工作流之前, 将所有的数据清理掉, 以便能够获取到最新的结果信息, 进行校验, 当然如果不清理, 也是没有任何问题的, 因为脚本之间对之前的所有数据进行覆盖操作
```



