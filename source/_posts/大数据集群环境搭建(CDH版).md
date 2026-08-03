---
title: 大数据集群环境搭建(CDH版)
date: 2024-04-27 00:00:00
author: Johns onLiam
tags: [CDH, 环境搭建]
top_img: https://wei-blog.oss-cn-beijing.aliyuncs.com/img/224634-171232839454f0.jpg
comments: true
---

# CDH集群搭建文档

# 一、环境要求

## 1.操作系统

### 1.1 软件依赖

-   Python 2.7+，不支持Python 3（安装HUE需要）
-   perl
-   python-psycopg2 2.5.4+（安装HUE需要）
-   iproute package
-   RHEL7：iproute-3.10
-   RHEL6：iproute-2.6

说明：通常这部分依赖环境操作系统已经自带，不需要手动安装。

### 1.2 系统版本

-   RHEL系列：6.10，6.9，6.8，7.2，7.3，7.4，7.5
-   SELS：12 SP3,12 SP2
-   Ubuntu系列：16.04 LTS

### 1.3 文件系统

-   ext3
-   ext4
-   XFS
-   S3

Kudu仅支持ext4和XFS文件系统。

不支持NFS和NAS存储。

### 1.4 文件访问记录

（挂载磁盘需要配置）

Cloudera建议禁用该项功能，可以提升磁盘性能。

配置/etc/fstab文件：

```properties
/dev/sdb1 /data1 ext4 defaults,noatime 0 0
```

不重启即生效命令：

```shell
mount -o remount /data1
```



### 1.5 nproc配置

> 配置文件位置/etc/security/limits.conf，Cloudera Manager通常会自动配置该文件，但是该项配置可能会被/etc/security/limits.d/目录下的文件覆盖掉。确保nproc限制设置的足够高，比如65536或者262144。

# 二、数据库要求

> Cloudera Manager和CDH内置嵌入式PostgreSQL数据库，用于非生产环境。生产环境不支持PostgreSQL数据库，必须为集群配置专用外部数据库。

注意：

-   数据库需要使用**UTF8**编码。对于**MySQL**和**MariaDB**，必须使用**UTF8**编码，而不是**utf8mb4**；
-   对于MySQL5.7，必须要额外安装**MySQL-shared-compat**或者**MySQL-shared**包

版本要求：

-   MySQL：5.7
-   MariaDB：5.5，10.0
-   PostgreSQL：8.4，9.2，9.4
-   Oracle：12C

# 三、Java要求

仅支持Oracle 64位JDK8，不支持JDK7，JDK9。

版本要求：最低要求1.8u31，推荐1.8u74，1.8u91，1.8u102，1.8u111，1.8u121，1.8u131，1.8u162

# 四、网络以及安全要求

-   CDH不支持IPv6，操作系统必须要禁用IPv6
    -   `setenforce 0`


-   集群主机必须正确配置/etc/hosts文件
-   应该包含集群中所有主机的主机名和对应IP地址
-   不能含有大写主机名
-   不能有重复的IP地址
-   SELinux不得阻止Cloudera Manager或者CDH的操作
-   防火墙必须被禁用或者放开CDH集群所使用的端口
-   对于RHEL或者CENTOS，/etc/sysconfig/network文件中必须包含正确的主机名

# 五、浏览器要求

-   Chrome：63+
-   Firefox：59+
-   Safari
-   IE：11+
-   Edge：41+

# 六、组件版本

| **组件名称**     | **组件版本** |
| ---------------- | ------------ |
| Apache Avro      | 1.8.2        |
| Apache Flume     | 1.8.0        |
| Apache Hadoop    | 3.0.0        |
| Apache HBase     | 2.0.0        |
| HBase Indexer    | 1.5          |
| Apache Hive      | 2.1.1        |
| Hue              | 4.2.0        |
| Apache Impala    | 3.0.0        |
| Apache Kafka     | 1.0.1        |
| Kite SDK         | 1.0.0        |
| Apache Kudu      | 1.6.0        |
| Apache Solr      | 7.0.0        |
| Apache Oozie     | 5.0.0        |
| Apache Parquet   | 1.9.0        |
| Parquet-format   | 2.3.1        |
| Apache Pig       | 0.17.0       |
| Apache Sentry    | 2.0.0        |
| Apache Spark     | 2.2.0        |
| Apache Sqoop     | 1.4.7        |
| Apache ZooKeeper | 3.4.5        |

# 七、安装之前

## 7.1 网络相关配置

-   设置唯一主机名：`hostnamectl set-hostname new-hostname`
-   配置`/etc/hosts`
-   检测主机名与IP是否正确对应

## 7.2 禁用防火墙

-   停止防火墙：`systemctl stop firewalld`
-   关闭防火墙开机自启动：`systemctl disable firewalld`

## 7.3设置SELinux执行模式

1.  检查SELinux模式：getenforce，如果输出**permissive**或者**disabled**，你可以跳过该步骤，如果输出**enforcing**，则需要继续下面的操作步骤。
2.  编辑/etc/selinux/config（在某些操作系统可能是/etc/sysconfig/selinux）文件，将SELINUX=enforcing修改为SELINUX=permissive，保存该文件。
3.  重启操作系统生效或者执行：setenforce 0临时禁用SELinux。
4.  当CDH安装部署完成之后，可以重新启用SELinux，修改SELinux配置文件，然后执行：setenforce 1命令。

## 7.4配置局域网内yum源(内网必备)

已有，可忽略此步骤

安装Cloudera Manager需要很多依赖包，强烈建议搭建一个局域网内yum源，在集群中某一个节点上部署即可。

国内有很多Yum的镜像源，比如阿里、网易等等，速度很快，使用着很方便。

但是，有些公司的生产环境是不能连接外网的，这样的环境下，不作一些措施的话，在CentOS上安装软件会很麻烦，依赖包的问题会很让人头疼。

所以最好是搭建一个局域网内yum仓库源。

### 7.4.1 准备工作

需要预先下载好Everything的CentOS7安装光盘包，Everything版的软件包比较全。

这里以CentOS7.6为例:

<http://mirrors.aliyun.com/centos/7.6.1810/isos/x86_64/>

### 7.4.2 配置FTP服务器

首先创建挂载目录

-   `mkdir /media/cdrom`
-   挂载ISO文件

使用VMware安装的虚拟机挂载方式，需要先确认好虚拟机已经连接上ISO文件：

![image-20240429220410575](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20240429220410575.png)

-   `mount -t auto /dev/sr0 /media/cdrom`

非CentOS7虚拟机，先上传ISO文件到某一目录，再挂载：

mount -t iso9660 -o loop /upload/[CentOS-7-x86_64-Everything-1810.iso](http://mirrors.aliyun.com/centos/7.6.1810/isos/x86_64/CentOS-7-x86_64-Everything-1810.iso) /media/cdrom

安装ftp软件（已安装的话就不用再安装了，可以使用`rpm -qa | grep vsftpd`命令检测）

-   `rpm -ivh /media/cdrom/Packages/vsftpd-3.0.2-28.el7.x86_64.rpm `
-   注意： 上述版本号会根据ISO文件的版本不同有所区别，Tab补全就行了。

启动vsftpd服务,并设为开机自启

- `systemctl start vsftpd`

查看21端口：

- `netstat -ntl | grep 21`

设为开机自启：

- `systemctl enable vsftpd`

浏览器访问ftp://ip_or_hostname

如果连接超时，说明开启了防火墙，需要关闭防火墙服务并关闭开机自启：

- `systemctl stop firewalld`

- `systwmctl disable firewalld`

- 如果连接需要账号密码:

  - `vim /etc/vsftpd/vsftpd.conf`
  
  ![descript](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/2bf24b8dc003d97f8c86c69e41b78d21.png)
  

部署yum仓库到ftp服务器

将安装光盘中的文件复制到ftp文件夹/var/ftp/pub/下

```shell
# 只复制两个目录(Packages,repodata)、一个文件(RPM-GPG-KEY-CentOS7)即可：
mkdir /var/ftp/pub/centos7
cp -rvf /media/cdrom/Packages /var/ftp/pub/centos7
cp -rvf /media/cdrom/repodata /var/ftp/pub/centos7
cp -rvf /media/cdrom/RPM-GPG-KEY-CentOS7 /var/ftp/pub/centos7

# 确认文件复制完毕以后就可以卸载光盘了
umount /media/cdrom

# 再次访问ftp://ip_or_hostname，就可以看到rpm包了
```



### 7.4.3配置yum仓库文件

备份原有的yum仓库文件

-   `cp -rvf /etc/yum.repo.d /upload/`

删除原有yum仓库配置文件

-   `rm -rf /etc/yum.repo.d/\*`

编辑新的yum仓库文件

```properties
cd /etc/yum.repo.d

vim ftp.repo

1.  [ftp]
2.  # 名字随便填
3.  name=ftp-repo
4.  # ftp服务器路径
5.  baseurl=ftp://ip_or_hostname/pub/centos7/
6.  # 1为启用GPG KEY检查，0禁用
7.  gpgcheck=0
8.  # 1为启用该仓库，0禁用
9.  enabled=1
10. # GPG KEY路径
11. gpgkey=ftp://ip_or_hostname/pub/centos7/RPM-GPG-KEY-CentOS-7
```



保存后,执行

-   `yum clean all`
    1.  Loaded plugins: fastestmirror, langpacks
    2.  Cleaning repos: ftp
    3.  Cleaning up everything
    4.  Cleaning up list of fastest mirrors

生成缓存

-   ` yum makecache`
    1.  Loaded plugins: fastestmirror, langpacks
    2.  ftp \| 3.6 kB 00:00:00
    3.  (1/4): ftp/group_gz \| 155 kB 00:00:00
    4.  (2/4): ftp/filelists_db \| 6.6 MB 00:00:00
    5.  (3/4): ftp/primary_db \| 5.6 MB 00:00:00
    6.  (4/4): ftp/other_db \| 2.4 MB 00:00:00
    7.  Determining fastest mirrors
    8.  Metadata Cache Created

然后就可以愉快的在内网环境下使用 `yum -y install package_name `来安装软件啦！！

## 7.5配置NTP服务

集群中所有节点的时间必须要保持同步，可以选择集群中其中一个节点作为ntp服务端，其余节点作为客户端。

-   安装ntp软件：yum -y install ntp
-   配置ntp服务端/etc/ntp.conf
-   配置ntp客户端/etc/ntp.conf
-   启动ntp服务：systemctl start ntpd
-   配置开机自启动：systemctl enable ntpd
-   同步客户端时间与服务端时间相同：ntpdate -u \<ntp-server\>

集群中所有节点必须保持时间同步，如果时间相差较大会引起问题(如HBase服务无法正常启动)

实现方法：master节点作为ntp服务器，对其它节点提供时间同步服务。 所有其它节点以master节点为基础同步时间。

1.所有节点安装相关ntp组件

\# yum install ntp/或者手动安装rpm包

2.所有节点设置时区,中国上海:

\# timedatectl set-timezone Asia/Shanghai

3.启动ntp，以及设置开机启动

\# systemctl start ntpd

\# systemctl enable ntpd

4.在master节点上设置现在的准确时间

\# date -s “2019-02-19 16:00:00”

5.配置ntp服务器(master1节点)

\# vim /etc/ntp.conf

配置示例：

![descript](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/affebedf9ab5e6332d70dc02fe958c27.png)

6.在其它节点上设置ntp客户端

\# vim /etc/ntp.conf

配置示例：

![image-20240428213300223](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20240428213300223.png)

7.配置文件修改完毕后，重启ntp服务

\# systemctl restart ntpd

8.在其它节点上手动同步master的时间

\# ntpdate -u 192.168.189.130

9.查看同步状态(可能需要稍等一会才能同步上)

\# ntpstat

![image-20240428213313022](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20240428213313022.png)

## 7.6 安装jdk8

> 集群所有节点都要安装。(JAVA_HOME必须是在/usr/java/否则找不到java_home)

1.查看系统自带java版本，如果已安装就先卸载掉

```shell
java -version
```



2.安装Oracle官网下载jdk的rpm安装包，并使用rpm -ivh packageName安装:

[Download jdk1.8](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)

3.修改环境变量

```shell
vim /etc/profile
```



添加如下(使用rpm安装的java在/usr/java/jdk1.8XXXX)

```properties
export JAVA_HOME=/usr/local/jdk1.8.0_241
export JRE_HOME=/usr/local/jdk1.8.0_241/jre
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$PATH
```



4.执行命令使环境变量生效

```shell
source /etc/profile
```



5.测试

```shell
java -version
```



![descript](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/6bdcfd1c33cce32dd7da7b04dc25eaaf.png)

## 7.7 安装mysql5.7

> 只需要在集群中某一个节点上安装即可。

文件准备：mysql-5.7.24-1.el7.x86_64.rpm-bundle.tar

![descript](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/1435c7bdc9d1fd1ff66aba085b5cbbc4.png)

- 首先卸载操作系统可能会自带的mariadb-libs

```shell
yum -y remove mariadb-libs
```



![descript](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/284c67dadaa04af579c2cb8b6657e891.png)

- 解压mysql rpm-bundle tar包

```shell
tar -xvf mysql-5.7.24-1.el7.x86_64.rpm-bundle.tar
```



- 开始安装mysql

- 一定要按照下面的顺序来安装，否则会安装不成功：

```shell
rpm -ivh mysql-community-common-5.7.24-1.el7.x86_64.rpm

rpm -ivh mysql-community-libs-5.7.24-1.el7.x86_64.rpm

rpm -ivh mysql-community-client-5.7.24-1.el7.x86_64.rpm

rpm -ivh mysql-community-server-5.7.24-1.el7.x86_64.rpm

rpm -ivh mysql-community-libs-compat-5.7.24-1.el7.x86_64.rpm（安装Cloudera Manager6需要）
```



![descript](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/063c9c6243d6a2ff8cde7fb912a2df17.png)

![descript](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/e0c65f0b302a1f831acaa335bc38cb3d.png)

- 启动mysql服务

```shell
systemctl start mysqld
```



- 如果无法启动，则需要修改mysql数据目录所有者：`chown -R mysql:mysql /var/lib/mysql/`

- 查看root用户初始密码

```shell
grep password /var/log/mysqld.log
```



![image-20240428213346058](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20240428213346058.png)

- 登录mysql修改root密码

```mysql
mysql -uroot -p

set password = password('123456');
```



![descript](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/7ec497d5646f8b8b3d67b438af6960c6.png)

如果密码复杂度不够，则会禁止修改，默认密码规则为：包含数字、大小写字母、特殊字符，同时还有长度要求：

```mysql
mysql> set password = password('123456');
ERROR 1819 (HY000): Your password does not satisfy the current policy requirements
```



可以通过修改全局参数来解决，但是还是要求密码长度至少为8位：

```mysql
mysql> set global validate_password_policy=0;
mysql> set password = password('12345678');
```



![descript](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/2b4e1c823c033509c7ce90b3528dd3b7.png)

设置远程登录权限

```mysql
grant all privileges on *.* to 'root'@'%' identified by '123456';

flush privileges;
```

![descript](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/8215fcc26b62c5a529ffc81a71459fe8.png)

- 如果上述设置远程登录权限出现错误

```properties
MySQL ERROR 3009 (HY000): Column count of mysql.user is wrong. Expected 45, found 42. Created with MySQL 50560, now running 50729. Please use mysql_upgrade to fix this error.
```

- 解决方案：
  - `mysql_upgrade -uroot -p`
- 报错原因：
  - 由于升级了数据库

- 修改mysql数据库默认编码

- 查看原数据库编码：`mysql> SHOW VARIABLES LIKE 'char%';`可以看到数据库和服务端的编码都还不是utf8：

![descript](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/01bbc21f090dd5aa7e9c65ebd71b3cd0.png)

- 如果上述查看数据库编码出现错误：

```properties
ERROR 1682 (HY000): Native table 'performance_schema'.'session_variables' has the wrong structure
```

- 解决方案：
  - `set @@global.show_compatibility_56=ON;`
- 报错原因：
  - 从mysql5.7.6开始information_schema.global_status已经开始被舍弃，为了兼容性，此时需要打开 show_compatibility_56

- 编辑/etc/my.cnf文件，在[mysqld]下面添加一行character-set-server=utf8；

```shell
vim /etc/my.cnf
# Notic ：下面的一句话必须在[mysqld]下面写
character-set-server=utf8

# 重启mysql服务：
systemctl restart mysqld
# 再次登录数据库查看编码，修改成功：
```



![descript](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/f5ea76b135c17f1972d3fcdb00a32661.png)

- 再次检查编码：

![descript](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/9801230105378d0586d16ff43ea5e0aa.png)

-   MySQL5.7安装配置完毕！

# 二、安装步骤

> 说明：所有操作都是在root用户下进行的。

## 2.1文件下载

首先一些安装CDH6集群的必须文件要先在外网环境先下载好。

### Cloudera Manager 6.0.1

CM6 RPM：https://archive.cloudera.com/cm6/6.0.1/redhat7/yum/RPMS/x86_64/

需要下载该链接下的所有RPM文件，由于jdk1.8我在环境准备部分已经手动安装了，所以可以不用下载RPMS/x86_64/目录下的jdk包oracle-j2sdk1.8-1.8.0+update141-1.x86_64.rpm，但是其他4个rpm包一定要下载，保存到cloudera-repos目录下。

ASC文件：<https://archive.cloudera.com/cm6/6.0.1/allkeys.asc>

同时还需要下载一个asc文件，同样保存到cloudera-repos目录下：

- `mkdir /upload/cloudera-repos -p`

![image-20240507205958835](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20240507205958835.png)

### MySQL JDBC驱动

要求使用5.1.26以上版本的jdbc驱动，可[点击这里](https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-5.1.47.tar.gz)直接下载mysql-connector-java-5.1.47.tar.gz

### CDH 6.0.1

CDH6 Parcels：https://archive.cloudera.com/cdh6/6.0.1/parcels/

需要下载CDH-6.0.1-1.cdh6.0.1.p0.590678-el7.parcel和manifest.json这两个文件

## 2.2配置Cloudera Manager yum库

```shell
# 注意：不要尝试使用FTP搭建CM的YUM库！

# 首先安装httpd和createrepo：

yum -y install httpd createrepo

# 启动httpd服务并设置开机自启动：

systemctl start httpd

systemctl enable httpd

然后进入到前面准备好的存放Cloudera Manager RPM包的目录cloudera-repos下：

cd /upload/cloudera-repos/
```



生成RPM元数据：

createrepo .

![descript](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/36744296c6e427751658ba9d7c44e927.png)

- 返回上级目录：
  - ` cd ..`
- 然后将cloudera-repos目录移动到httpd的html目录下：

```shell 
mv cloudera-repos /var/www/html/
```

确保可以通过浏览器查看到这些RPM包：

我这里浏览器地址为：

```properties
http://192.168.88.161/cloudera-repos/
```



![descript](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/28b8c23fd02a4808e26f3a1941160225.png)

接着在Cloudera Manager Server主机上创建cm6的repo文件（要把哪个节点作为Cloudera Manager Server节点，就在这个节点上创建repo文件）：

```shell
cd /etc/yum.repos.d

vim cloudera-manager.repo
```



添加如下内容：

```properties
[cloudera-manager]
name=Cloudera Manager 6.0.1
baseurl=http://192.168.88.161/cloudera-repos/
gpgcheck=0
enabled=1
```

![descript](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/b65161c6423dbc35580e4b680ec6b3db.png)

保存，退出,然后执行`yum clean all && yum makecache`命令：

![descript](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/c0a573303b9b55ec2d8444eab2081468.png)

## 2.3安装Cloudera Manager Server

这一步只需要在CM Server节点上操作。

执行下面的命令：

`yum install cloudera-manager-daemons cloudera-manager-agent cloudera-manager-server`

将会需要很多依赖包，所以说还是有必要搭一个局域网内yum源的：

![descript](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/8dd733642f256d20c1caf99ef046caaf.png)

## 2.4配置本地Parcel存储库

```shell
# Cloudera Manager Server安装完成后，进入到本地Parcel存储库目录：

cd /opt/cloudera/parcel-repo

# 将第一部分下载的CDH Parcel文件（CDH-6.0.1-1.cdh6.0.1.p0.590678-el7.parcel和manifest.json）上传至该目录下，然后执行命令生成sha文件：
cp /var/www/html/cloudera-repos/CDH-6.0.1-1.cdh6.0.1.p0.590678-el7.parcel /opt/cloudera/parcel-repo/

sha1sum CDH-6.0.1-1.cdh6.0.1.p0.590678-el7.parcel | awk '{ print $1 }' > CDH-6.0.1-1.cdh6.0.1.p0.590678-el7.parcel.sha

# 然后执行下面的命令修改文件所有者：

chown -R cloudera-scm:cloudera-scm /opt/cloudera/parcel-repo/*
```



最终/opt/cloudera/parcel-repo目录内容如下：

![descript](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/9a07de01ec4d862f1f8cf5d80c6e67ef.png)

## 2.5 安装数据库

MySQL的安装在环境准备部分中已经有说明，这里就跳过MySQL安装了。

```shell
vim /etc/my.cnf
```

### 数据库配置

```properties
# For advice on how to change settings please see
# http://dev.mysql.com/doc/refman/5.7/en/server-configuration-defaults.html

[mysqld]
#
# Remove leading # and set to the amount of RAM for the most important data
# cache in MySQL. Start at 70% of total RAM for dedicated server, else 10%.
# innodb_buffer_pool_size = 128M
#
# Remove leading # to turn on a very important data integrity option: logging
# changes to the binary log between backups.
# log_bin
#
# Remove leading # to set options mainly useful for reporting servers.
# The server defaults are faster for transactions and fast SELECTs.
# Adjust sizes as needed, experiment to find the optimal values.
# join_buffer_size = 128M
# sort_buffer_size = 2M
# read_rnd_buffer_size = 2M
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
character-set-server=utf8
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0
#skip_grant_tables
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid

sql_mode = STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION

max_connections=50000

[client]
default-character-set=utf8

[mysql]
default-character-set=utf8

```

![image-20240428213517263](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20240428213517263.png)

## 2.6配置mysql jdbc驱动

```shell
从前面下载好的mysql-connector-java-5.1.47.tar.gz包中解压出mysql-connector-java-5.1.47-bin.jar文件，将mysql-connector-java-5.1.47-bin.jar文件上传至CM Server节点上的/usr/share/java/目录下并重命名为mysql-connector-java.jar（如果/usr/share/java/目录不存在，需要手动创建）：

tar zxvf mysql-connector-java-5.1.47.tar.gz

mkdir -p /usr/share/java/

cd mysql-connector-java-5.1.47

cp mysql-connector-java-5.1.47-bin.jar /usr/share/java/mysql-connector-java.jar
```



## 2.7创建CDH所需要的数据库

根据所需要安装的服务参照下表创建对应的数据库以及数据库用户，数据库必须使用utf8编码，创建数据库时要记录好用户名及对应密码：

| **服务名**                         | **数据库名** | **用户名** |
| ---------------------------------- | ------------ | ---------- |
| Cloudera Manager Server            | scm          | scm        |
| Activity Monitor                   | amon         | amon       |
| Reports Manager                    | rman         | rman       |
| Hue                                | hue          | hue        |
| Hive Metastore Server              | metastore    | hive       |
| Sentry Server                      | sentry       | sentry     |
| Cloudera Navigator Audit Server    | nav          | nav        |
| Cloudera Navigator Metadata Server | navms        | navms      |
| Oozie                              | oozie        | oozie      |

我这里就先创建4个数据库及对应用户：

```mysql
create database scm default character set utf8 default collate utf8_general_ci;

create database amon default character set utf8 default collate utf8_general_ci;

create database rman default character set utf8 default collate utf8_general_ci;

create database metastore default character set utf8 default collate utf8_general_ci;

grant all on scm.* to 'scm'@'%' identified by 'scm';

grant all on amon.* to 'amon'@'%' identified by 'amon';

grant all on rman.* to 'rman'@'%' identified by 'rman';

grant all on metastore.* to 'hive'@'%' identified by 'hive';

flush privileges;
```



![descript](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/5b3947580eda778fd2716fc192502374.png)

注意 要是用5.7.tar.gz安装包安装的mysql用以上命令创建数据库回报语法错误

![descript](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/0dbfb35584699298a5bb6582c31ea5d4.png)

用一下语法格式

![descript](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/61d4f13023a94f180406fda16bfbdf31.png)

查看授权是否正确：

```mysql
show grants for  'scm'@'%';

show grants for  'amon'@'%';

show grants for 'rman'@'%';

show grants for  'hive'@'%';

```

![descript](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/dda3db0f551ce201ce87a73e7d87587c.png)

## 2.8设置Cloudera Manager 数据库

> 在这一步区分mysql是否和cdh安装在一台服务器上面的。

Cloudera Manager Server包含一个配置数据库的脚本。

- 如果mysql数据库与CM Server是同一台主机

  - 执行命令：`/opt/cloudera/cm/schema/scm_prepare_database.sh mysql scm scm`
  - 上述命令执行之后，需要输入scm用户的密码：直接写scm就行

  - ![image-20240512151957455](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20240512151957455.png)
  - 类似于：![image-20240512151939224](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20240512151939224.png)

![descript](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/27d767c437942a3fb1fb0f8ddafd09ed.png)

- 如果mysql数据库与CM Server不在同一台主机上
  - 执行命令：/opt/cloudera/cm/schema/scm_prepare_database.sh mysql -h 192.168.88.161 --scm-host 192.168.88.161 scm scm
  - 将上述ip换成CM 主节点ip地址

## 2.9安装CDH节点

### 启动Cloudera Manager Server服务

`systemctl start cloudera-scm-server`

然后等待Cloudera Manager Server启动，可能需要稍等一会儿，可以通过命令

`tail -f /var/log/cloudera-scm-server/cloudera-scm-server.log`去监控服务启动状态。

当看到INFO WebServerImpl:com.cloudera.server.cmf.WebServerImpl: Started Jetty server.日志打印出来后，说明服务启动成功，可以通过浏览器访问Cloudera Manager WEB界面了。

### 访问Cloudera Manager WEB界面

打开浏览器，访问地址：http://192.168.88.161:7180，默认账号和密码都为admin：

![descript](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/75c5186e7b398265708b024fcb7cb650.png)

### 欢迎页面

首先是Cloudera Manager的欢迎页面，点击页面右下角的【继续】按钮进行下一步：

![descript](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/32beefb11552064e4218e4d7723e1f28.png)

### 接受条款

勾选接受条款，点击【继续】进行下一步：

![descript](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/6fa819f6b41d12eef3ffe46a6822941d.png)

### 版本选择

这里我就选择免费版了：

![descript](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/7a47f48976edd00642e421d3247429b2.png)

### 第二个欢迎界面

选择版本以后会出现第二个欢迎界面，不过这个是安装集群的欢迎页：

![descript](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/532ee042e5e6f97b9e9bae6009d8a559.png)

### 选择主机

这一步是要搜索并选择用于安装CDH集群的主机，在主机名称后面的输入框中输入各个节点的hostname，中间使用英文逗号分隔开，然后点击搜索，在结果列表中勾选要安装CDH的节点即可：

![descript](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/3947d892cedf0d3fc2728fe5a4dfc1cb.png)

### 指定存储库

#### Cloudera Manager Agent

这里选择自定义，填写上面使用httpd搭建好的Cloudera Manager YUM 库URL：

![descript](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/a422100f6d1769024afa18b3f05b5c93.png)

#### CDH and other software

如果我们之前的【配置本地Parcel存储库】步骤操作无误的话，这里会自动选择【使用Parcel】，并加载出CDH版本，确认无误后点击【继续】：

![descript](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/08063f331a099885f3e07c7cb8640b34.png)

### JDK安装选项

这一步骤我就不再勾选安装JDK了，因为我在环境准备部分已经安装过了。取消勾选，然后继续：

![descript](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/f67c2bd1ef1079fecccaa4139c4fe1e6.png)

### SSH登录配置

用于配置集群主机之间的SSH登录，填写root用户的密码，根据集群配置填写合适的【同时安装数量】值即可：

![descript](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/ed97a2951b9e7038cd8225f5621e288e.png)

### 安装Agent

到这一步会自动进行节点Agent的安装，稍等一会儿，即可安装完成：![descript](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/fac2c2e917e7aca08714e03f81922709.png)

![descript](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/007b964affb3f7230f540fee3b577e53.png)

### 安装Parcels

这一步同样是自动安装，分配步骤的速度主要取决于网络环境，耐心等待即可（我的3台虚拟机性能实在是太差了，这一步等了好久）：![descript](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/5de08837df54a2a5eaf6e1d09d8fcdc6.png)

![descript](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/b0574ea5202ed581e395478b4b9ab088.png)

### 主机检查

等待检查完成即可：

![descript](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/960122d3469bad3419d238eca146fc69.png)

## 2.10安装CDH集群

### 选择服务类型

这里我选择自定义服务，HDFS，Hive，Yarn：

![descript](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/981e9fb767753938e414d8a1ad36f4dd.png)

### 角色分配

CDH会自动给出一个角色分配，如果觉得不合理，我们可以手动调整一下，注意角色分配均衡：

![image-20240428213550324](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/image-20240428213550324.png)

### 数据库设置

因为我选择的服务中只有Hive需要数据库，故这里只需要配置Hive的metastore数据库。注意要将mysql的jdbc驱动放到hive metastore主机的/usr/share/java/目录下：

![descript](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/a021cdb637db791d82c578840e11985f.png)

到这里CDH集群的安装基本上就已经完成了。

**2022年4月20日新增**

# 在CDH中添加flume组件

1.在安装好CDH页面后，切换到首页，点击集群名称右侧的下拉箭头，下拉菜单中选择 添加服务

![descript](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/bc1f563ae519cd3a5fb17489e3cca54f.png)

2\. 在新的页面中选中 Flume 服务

![descript](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/ec0140d892fbcf2d1e1f5bdbda096597.png)

3\. 页面下拉到最后，选择继续

![descript](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/f5805376f62f22f717fa24d9f0e39a37.png)

4\. 在新的页面中点击 选择主机

![descript](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/bd7f5588e0fc5a8f9350b882209ded25.png)

5\. 选择要安装flume组件的主机，然后点击确定。

主机选择时，请根据实际情况，选择已经配置syslog程序的一台主机，此处里slave4为例。

![descript](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/f6de4f7bba7a8b6af5e74bdb091720e8.png)

6\. 确认主机选择后，点击继续

![descript](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/feb3831a32958eba60fe58a62b887877.png)

7 继续点击完成，将跳回首页

![descript](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/66e3a70c117d7423f4f66538a346078c.png)

![descript](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/7cd71d869448819156eb3ea6588b03f8.png)

8\. 点击选择新安装的Flume

演示是第二次安装flume，所有新装的flume组件名称是Flume-2

如果是第一次安装flume，那么名称是Flume，请注意辨别

![descript](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/2746fe495fcf1894c75b3331a68f369d.png)

9.进入页面后，选择配置

![descript](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/0f825da29d599009cdb08d4f1b7b1b4f.png)

10\. 在配置页面中找到配置文件选项，将当前的配置文件内容情况，将已经配置好的配置文件（flume.properties）内容复制进去

![descript](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/4b554b2d9cdb5ef9dafe38955b988558.png)

11\. 配置修改之后，点击保存更改。

![descript](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/3a0e515b98f9fd85a47663e5c28a44ca.png)

12.安装flune拦截器Flumeinterceptor-1.0-SNAPSHOT.jar（在CDH\\service\\flume\\flumeD下）

SSH登录到安装flume节点的服务器，将jar文件复制到 /opt/cloudera/parcels/CDH-6.0.1-1.cdh6.0.1.p0.590678/lib/flume-ng/lib 下

13\. 启动flume，待启动之后查看状态，或查看日志(对应主机 /var/log/flume/)

![descript](https://wei-blog.oss-cn-beijing.aliyuncs.com/img/1b5970a95c1fd8d935111082ca28c279.png)

![descript](C:\Users\admin\Desktop\opt\Document\e2ffd14b0789708fbc422904687f7b31.png)





