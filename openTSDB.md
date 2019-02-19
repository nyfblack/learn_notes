# 1.CentOS7环境配置

## 1.CentOS7 配置 JAVA 环境

### 1.  卸载自带的openjdk

* 1.查看当前jdk相关内容

  ~~~shell
  rpm -qa | grep java
  ~~~

* 2.如果出现以下内容

  ~~~shell
  java-1.7.0-openjdk-1.7.0.9-2.3.4.1.el6_3.x86_64
  java-1.6.0-openjdk-1.6.0.0-1.50.1.11.5.el6_3.x86_64
  java-1.6.0-openjdk-devel-1.6.0.0-1.50.1.11.5.el6_3.x86_64
  java-1.7.0-openjdk-devel-1.7.0.9-2.3.4.1.el6_3.x86_64
  ~~~

* 3.删除2中出现的内容

  ~~~shell
  rpm -e --nodeps xxx
  rpm -e --nodeps java-1.7.0-openjdk-1.7.0.9-2.3.4.1.el6_3.x86_64
  rpm -e --nodeps java-1.6.0-openjdk-1.6.0.0-1.50.1.11.5.el6_3.x86_64
  ~~~

### 2. 安装 JDK

* 1.从Oracle官网下载对应的JDK包[jdk下载地址]("http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html"),当前我使用的centos是64位所以下载的为:

  | **Product / File Description** | **File Size** | **Download**              |
  | ------------------------------ | ------------- | ------------------------- |
  | Linux x64                      | 185.05 MB     | dk-8u202-linux-x64.tar.gz |

* 2.上传JDK安装包至Centos
  ~~~shell
  cd 
  mkdir /usr/java
  mkdir /repo
  ~~~

  使用WinSCP或者FileZilla将JDK安装包上传至Centos

* 3.解压JDK安装包
  ~~~shell
  mv /repo/jdk-8u202-linux-x64.tar.gz /usr/java
  tar -zxvf jdk-8u202-linux-x64.tar.gz
  ~~~

* 4.配置环境变量

  ~~~shell
  vim /etc/profile
  ~~~

  在文件的末尾添加如下配置,其中JDK的版本要与自己安装的对应

  ~~~shell
  #set java enviroment 
  JAVA_HOME=/usr/java/jdk1.8.0_202
  JRE_HOME=/usr/java/jdk1.8.0_202/jre
  CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
  PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
  export JAVA_HOME JRE_HOME CLASS_PATH PATH
  ~~~

  设置完成后,退出vim,使用如下指令刷新环境变量

  ~~~shell
  source /etc/profile
  ~~~

* 5.验证是否安装成功

  ~~~shell
  java -version
  ~~~


## 2.CentOS7 Zookeeper伪集群安装

### 1. Zookeeper简介

&ensp;&ensp;ZooKeeper是一个分布式的，开放源码的分布式应用程序协调服务，是Google的Chubby一个开源的实现，是Hadoop和Hbase的重要组件。它是一个为分布式应用提供一致性服务的软件，提供的功能包括：配置维护、域名服务、分布式同步、组服务等。

### 2. 伪集群模式安装过程

* 从[zookeeper](http://mirror.bit.edu.cn/apache/zookeeper/)官网下载zookeeper安装包或者直接在Linux中使用wget命令下载

  ~~~shell
  wget http://mirror.bit.edu.cn/apache/zookeeper/zookeeper-3.4.13/zookeeper-3.4.13.tar.gz
  ~~~

* 上传安装包至linux服务器(使用wget可以跳过此步骤)

* 解压缩安装包

  ~~~shell
  mkdir -p /usr/zookeeper
  mv /repo/zookeeper-3.4.13.tar.gz /usr/zookeeper
  tar -zxvf /user/zookeeper/zookeeper-3.4.13.tar.gz
  ~~~

* 创建data目录和logs目录

  ~~~shell
  cd /usr/zookeeper
  mkdir -p data/zk1
  mkdir -p data/zk2
  mkdir -p data/zk3
  mkdir -p logs/zk1
  mkdir -p logs/zk2
  mkdir -p logs/zk3
  ~~~

* 修改配置文件

  ~~~shell
  cd /usr/zookeeper/zookeeper-3.4.13/conf
  ~~~

  ~~~shell
  mv zoo-simple.cfg zoo1.cfg
  vi zoo1.cfg
  #修改如下配置
  clientPort=2181
  dataDir=/usr/zookeeper/data/zk1
  dataLogDir=/usr/zookeeper/logs/zk1
  server.1=localhost:2887:3887
  server.2=localhost:2888:3888
  server.3=localhost:2889:3889
  ~~~

  ~~~shell
  cp zoo1.cfg zoo2.cfg
  vi zoo2.cfg
  #修改如下配置
  clientPort=2182
  dataDir=/usr/zookeeper/data/zk2
  dataLogDir=/usr/zookeeper/logs/zk2
  server.1=localhost:2887:3887
  server.2=localhost:2888:3888
  server.3=localhost:2889:3889
  ~~~

  ~~~shell
  cp zoo1.cfg zoo3.cfg
  vi zoo3.cfg
  #修改如下配置
  clientPort=2183
  dataDir=/usr/zookeeper/data/zk3
  dataLogDir=/usr/zookeeper/logs/zk3
  server.1=localhost:2887:3887
  server.2=localhost:2888:3888
  server.3=localhost:2889:3889
  ~~~

* 配置myid

  ~~~shell
  cd /usr/zookeeper
  echo 1 > data/zk1/myid
  echo 2 > data/zk2/myid
  echo 3 > data/zk3/myid
  ~~~

* 关闭防火墙

  ~~~shell
  systemctl start firewalld #启动
  systemctl stop firewalld #关闭
  systemctl status firewalld #查看状态 
  systemctl disable firewalld #开机禁用
  systemctl enable firewalld #开机启用
  ~~~

* 启动服务或者关闭服务

  ~~~shell
  #开启服务
  cd /user/zookeeper/zookeeper-3.4.13/bin
  sh ./zkServer.sh start ../conf/zoo1.cfg
  sh ./zkServer.sh start ../conf/zoo2.cfg
  sh ./zkServer.sh start ../conf/zoo3.cfg
  #关闭服务
  sh ./zkServer.sh stop ../conf/zoo1.cfg
  sh ./zkServer.sh stop ../conf/zoo2.cfg
  sh ./zkServer.sh stop ../conf/zoo3.cfg
  #查看服务状态
  sh ./zkServer.sh status ../conf/zoo1.cfg
  sh ./zkServer.sh status ../conf/zoo2.cfg
  sh ./zkServer.sh status ../conf/zoo3.cfg
  ~~~

* 将zookeeper注册为系统服务,方便使用

  ~~~shell
  cd /etc/init.d
  vi zookeeper
  ~~~

  ~~~shell
  ROOT_PATH=/user/zookeeper/zookeeper-3.4.13
  
  case "$1" in
  
  start)
  echo "Starting zookeeper Server...."
  sh $ROOT_PATH/bin/zkServer.sh start $ROOT_PATH/conf/zoo1.cfg
  sh $ROOT_PATH/bin/zkServer.sh start $ROOT_PATH/conf/zoo2.cfg
  sh $ROOT_PATH/bin/zkServer.sh start $ROOT_PATH/conf/zoo3.cfg
  echo "Starting zookeeper Server success!!!"
  ;;
  
  stop)
  echo "Stop zookeeper Server..."
  sh $ROOT_PATH/bin/zkServer.sh stop $ROOT_PATH/conf/zoo1.cfg
  sh $ROOT_PATH/bin/zkServer.sh stop $ROOT_PATH/conf/zoo2.cfg
  sh $ROOT_PATH/bin/zkServer.sh stop $ROOT_PATH/conf/zoo3.cfg
  echo "Stop zookeeper Server success!!!"
  ;;
  
  status)
  sh $ROOT_PATH/bin/zkServer.sh status $ROOT_PATH/conf/zoo1.cfg
  sh $ROOT_PATH/bin/zkServer.sh status $ROOT_PATH/conf/zoo2.cfg
  sh $ROOT_PATH/bin/zkServer.sh status $ROOT_PATH/conf/zoo3.cfg
  ;;
  
  esac
  
  exit 0
  ~~~

  通过上面的脚本,就可以方便的通过service命令来管理zookeeper了

  ~~~shell
  service zookeeper start #开启服务
  service zookeeper status #查看服务状态
  service zookeeper stop #停止服务
  ~~~

### 3. 安装过程中遇到的问题

​	在启动服务后使用./zkServer.sh status查看服务状态时,一直报"Error contacting service. It is probably not running",查看服务启动目录下的zookeeper.out文件发现是"java.net.ConnectException: 拒绝连接"原因,排查发现是因为之前zookeeper异常停止导致的,将每个节点**data**目录下的**zookeeper_server.pid**文件删除,然后重启服务即可.

​	上述问题还有可能是端口占用,配置出错,没有关闭防火墙导致的,具体原因,逐步排查就能发现.



## 3.CentOS7 Hbase伪集群安装

### 1. Hbase简介

&ensp;&ensp;HBase是一个分布式的、面向列的开源数据库，该技术来源于 Fay Chang 所撰写的Google论文“Bigtable：一个结构化数据的分布式存储系统”。就像Bigtable利用了Google文件系统（File System）所提供的分布式数据存储一样，HBase在Hadoop之上提供了类似于Bigtable的能力。HBase是Apache的Hadoop项目的子项目。HBase不同于一般的关系数据库，它是一个适合于非结构化数据存储的数据库。另一个不同的是HBase基于列的而不是基于行的模式。

### 2. Hbase安装过程

&ensp;&ensp;**Hbase 需要 JAVA 运行环境**,所以安装Hbase之前请确保当前系统中已经安装有JAVA环境;

* 1.从[Hbase官网](http://hbase.apache.org)下载[Hbase安装包](http://mirrors.hust.edu.cn/apache/hbase/2.1.3/hbase-2.1.3-bin.tar.gz)

* 2.使用WinScp或者FileZilla将Hbase安装包上传至服务器

* 3.解压缩安装包

  ~~~shell
  mkdir -p /usr/hbase
  mv /repo/hbase-2.1.3-bin.tar.gz /usr/hbase
  tar -zxvf /usr/hbase/hbase-2.1.3-bin.tar.gz
  ~~~

* 4.创建保存数据文件夹

  ~~~shell
  mkdir -p /usr/hbase/data
  ~~~

* 5.配置hbase-env.sh

  ~~~shell
  cd /usr/hbase/hbase-2.1.3/conf
  vi hbase-env.sh
  ~~~

  在末尾添加如下内容:

  ~~~shell
  export JAVA_HOME=/usr/java/jdk1.8.0_202           #本机JAVA安装路径
  export HBASE_MANAGES_ZK=false #不使用自带zookeeper
  ~~~

* 6.配置hbase-site.xml文件

  &ensp;&ensp;这是HBase的主配置文件。在hbase-site.xml文件里面，找到 <configuration>  标签。并在其中，设置属性键名为“hbase.rootdir”。 设置数据保存的目录：

  ~~~shell
  vi hbase-site.xml
  ~~~

  ~~~xml
  <configuration>
    <property>
      <name>hbase.rootdir</name>
      <value>/usr/hbase/data</value>
    </property>
  </configuration>
  ~~~

* 7.Hbase启动和停止

  ~~~shell
  sh /usr/hbase/hbase-2.1.3/bin/start-hbase.sh #启动hbase
  sh /usr/hbase/hbase-2.1.3/bin/stop-hbase.sh #停止hbase
  ~~~

### 3. Hbase简单使用

**在bin目录中输入./hbase shell进入hbase客户端** 

~~~shell
hbase(main):002:0> create 'test', 'cf'
Created table test
Took 1.2630 seconds
=> Hbase::Table - test
hbase(main):003:0> list
TABLE
test
1 row(s)
Took 0.0400 seconds
=> ["test"]
hbase(main):004:0> put 'test', 'row1', 'cf:a', 'value1'
Took 0.3619 seconds
hbase(main):005:0> put 'test', 'row2', 'cf:b', 'value2'
Took 0.0264 seconds
hbase(main):006:0> put 'test', 'row3', 'cf:c', 'value3'
Took 0.0228 seconds
hbase(main):007:0> scan 'test'
ROW                                 COLUMN+CELL
 row1                               column=cf:a, timestamp=1550302844652, value=value1
 row2                               column=cf:b, timestamp=1550302862478, value=value2
 row3                               column=cf:c, timestamp=1550302890273, value=value3
3 row(s)
Took 0.0514 seconds
hbase(main):008:0> get 'test', 'row1'
COLUMN                              CELL
 cf:a                               timestamp=1550302844652, value=value1
1 row(s)
Took 0.0488 seconds
hbase(main):009:0> create 'test2', 'cf'
Created table test2
Took 0.7756 seconds
=> Hbase::Table - test2
hbase(main):010:0> list
TABLE
test
test2
2 row(s)
Took 0.0188 seconds
=> ["test", "test2"]

~~~

## 4.CentOS7 OpenTSDB伪集群安装 

### 1. OpenTSDB简介

&ensp;&ensp;OpenTSDB是基于HBase存储时间序列数据的一个开源数据库，但只是一个HBase的应用而已。也即是在HBase之上加了一层外壳，用于更好的处理时序数据库，真实的数据存储还是在HBase。
&ensp;&ensp;时序数据是基于时间的一系列的数据。在有时间的坐标中将这些数据点连成线，往过去看可以做成多纬度报表，揭示其趋势性、规律性、异常性；往未来看可以做大数据分析，机器学习，实现预测和预警。时序数据库就是存放时序数据的数据库，并且需要支持时序数据的快速写入、持久化、多纬度的聚合查询等基本功能。
&ensp;&ensp;OpenTSDB实现的时间序列Schema主要有两个表：tsdb-uid和tsdb. 前者描述指标（metrics）相关的元数据，后者存储时间序列数据。首先我们来了解一下“指标”（metrics）的概念，简单讲一个指标就是一个需要收集的数据项，但是只有指标是不能全面地描述出一条数据产生的相关背景信息的，比如：如果我们要统计cpu的使用率，我们可以建立一下名为proc.stat.cpu的metrics,如果我们从不同的机器和用户下收集了大量的cpu信息，如果没有对一条信息进行一定地标识，我们是无法区分出哪些数据来自哪台机器的哪个用户，所以我们还需要建立一些“标签”（Tag）来标识一条数据。严格地说，指标和标签之间并没有必然的从属关系，就像两个不同的指标的数据可能都有指示其来自哪台主机的host标签一样，但是有一点是确定的，即：对于一条数据来说，应该至少含有一个指标和一个标签，这样的数据才是有意义的，因此，在OpenTSDB的表设计上，就把“指标”（metrics）和“标签”（Tag）统一放在了tsdb-uid表中存储，格式为：RowKey(自增ID，3字节数组）name:metrics,name:tagk,name:tagv，同时对它们之间的反向关联关系也作了展开存储。

### 2. OpenTSDB安装过程

* 安装Hbase,可以参考Centos7 安装Hbase

* 从[OpenTSDB官网](https://github.com/OpenTSDB/opentsdb/releases)下载OpenTSDB安装包(github上的)

* 使用WinSCP或者Filezilla将安装包上传至Linux

* 解压缩安装包

  ~~~shell
  mkdir -p /usr/opentsdb
  mv /repo/opentsdb-2.4.0.tar.gz /usr/opentsdb
  cd /usr/opentsdb
  tar -zxvf opentsdb-2.4.0.tar.gz
  ~~~

* 编译源文件,生成tsdb-2.4.0.jar

  ~~~shell
  cd opentsdb-2.4.0
  ./configure
  make
  ~~~

* 创建schema所需要的表

  ~~~shell
  env COMPRESSION=NONE HBASE_HOME=/usr/hbase/hbase-2.1.3 ./src/create_table.sh
  ~~~

  **HBASE_HOME指的是本机Hbase的安装目录**

* 修改配置文件opentsdb.conf,该文件可以从./src文件得到,将该文件移至opentsdb根目录

  ~~~shell
  mv ./src/opentsdb.conf ./
  vi opentsdb.conf
  ~~~

  ~~~shell
  # 接受请求的端口
  tsd.network.port = 4399
  
  # 接受请求的网卡
  tsd.network.bind = 0.0.0.0
  
  # HTTP客户端的GUI静态页面，这个使用默认值即可。
  tsd.http.staticroot = ./staticroot
  
  # cache 路径，最好提前创建好，保证读写权限。
  tsd.http.cachedir = /usr/opentsdb/tsdb_cache
  
  # 是否能自动创建统计指标
  tsd.core.auto_create_metrics = true
  
  # HBase表名称
  tsd.storage.hbase.data_table = tsdb
  
  # HBase访问相关
  tsd.storage.hbase.zk_basedir = /hbase
  tsd.storage.hbase.zk_quorum = localhost
  tsd.storage.fix_duplicates = true
  ~~~

* 启动tsdb服务

  ~~~shell
  cd /user/opentsdb/opnetsdb-2.4.0
  ./tsdb tsd
  ~~~

* 使用浏览器访问opentsdb.访问地址**ip:4399**




# 2.OpenTSDB原理

## 1.元数据

### 1.什么是时序数据?

> *时间序列**（Time Series）是一组按照时间发生先后顺序进行排列的数据点序列**，通常一组时间序列的时间间隔为一恒定值（如1秒，5分钟，1小时等）。*

### 2.时序数据的特点

实时监控系统所收集的监控指标数据，通常就是时序数据 。时序数据具有如下特点：

* 每一个时间序列通常为某一固定类型的数值
* 数据按一定的时间间隔持续产生，每条数据拥有自己的时间戳信息
* 通常只会不断的写入新的数据，几乎不会有更新、删除的场景
* 在读取上，也往往倾向于读取最近写入的数据。

### 3.OpenTSDB中的时序数据

如下是OpenTSDB官方资料中的时序数据样例

>sys.cpu.user host=webserver01 1356998400 50
>
>sys.cpu.user host=webserver01,cpu=0 1356998400 1
>
>sys.cpu.user host=webserver01,cpu=1 1356998400 0
>
>sys.cpu.user host=webserver01,cpu=2 1356998400 2
>
>sys.cpu.user host=webserver01,cpu=3 1356998400 0
>
>…………
>
>sys.cpu.user host=webserver01,cpu=63 1356998400 1

对于**上面的任意一行数据，在OpenTSDB中称之为一个时间序列中的一个Data Point**。以最后一行为例说明一下OpenTSDB中关于Data Point的每一部分组成定义如下：

| 构成信息     | 名称      |
| ------------ | --------- |
| sys.cpu.user | metrics   |
| host         | tagKey    |
| webserver01  | tagValue  |
| cpu          | tagKey    |
| 63           | tagValue  |
| 1356998400   | timestamp |
| 1            | value     |

可以看出来，每一个Data Point，都关联一个metrics名称，但可能关联多组<tagKey,tagValue>信息。而关于时间序列，事实上就是具有相同的metrics名称以及相同的<tagKey,tagValue>组信息的Data Points的集合。在存储这些Data Points的时候，大家也很容易可以想到，可以将这些metrics名称以及<tagKey,tagValue>信息进行特殊编码来优化存储，否则会带来极大的数据冗余。OpenTSDB中为每一个metrics名称，tagKey以及tagValue都定义了一个唯一的数字类型的标识码(UID)。

### 4.OpenTSDB中的元数据模型

1. **UID**

   UID的全称为Unique Identifier。这些UID信息被保存在OpenTSDB的元数据表中，默认表名为”tsdb-uid”。

   OpenTSDB分配UID时遵循如下规则：

* metrics、tagKey和tagValue的UID分别独立分配
* 每个metrics名称（tagKey/tagValue）的UID值都是唯一。不存在不同的metrics（tagKey/tagValue）使用相同的UID，也不存在同一个metrics（tagKey/tagValue）使用多个不同的UID
* UID值的范围是0x000000到0xFFFFFF，即metrics（或tagKey、tagValue）最多只能存在16777216个不同的值。

2. **关于元数据在HBase中的表设计**

   为了从UID索引到metrics（或tagKey、tagValue），同时也要从metrics（或tagKey、tagValue）索引到UID，OpenTSDB同时保存这两种映射关系数据。在元数据表中，把这两种数据分别保存到两个名为”id”与”name”的Column Family中，Column Family描述信息如下所示：

> {NAME => ‘id’, BLOOMFILTER => ‘ROW’, COMPRESSION => ‘SNAPPY’}
>
> {NAME =>’name’,BLOOMFILTER => ‘ROW’, COMPRESSION => ‘SNAPPY’, MIN_VERSIONS => ‘0’, BLOCKCACHE => ‘true’, BLOCKSIZE => ‘65536’, REPLICATION_SCOPE => ‘0’}

3. **元数据模型**

   关于metrics名为”cpu.hum”，tagKey值为”host”，tagValue值分别为”189.120.205.26″、”189.120.205.27″的UID信息定义如下:

| RowKey         | metrics     | tagK        | tagv           | metrics | tagk | tagv |
| -------------- | ----------- | ----------- | -------------- | ------- | ---- | ---- |
| 0              | UID_COUNTER | UID_COUNTER | UID_COUNTER    | --      | --   | --   |
| 1              | cpu.hum     | host        | 189.120.205.26 | --      | --   | --   |
| 2              | --          | --          | 189.120.205.26 | --      | --   | --   |
| 189.120.205.26 | --          | --          | --             | --      | --   | 1    |
| 189.120.205.27 | --          | --          | --             | --      | --   | 2    |
| cpu.hum        | --          | --          | --             | 1       | --   | --   |
| host           | --          | --          | --             | --      | 1    | --   |

**说明:**

* RowKey为”0″的行中，分别保存了metrics、tagKey和tagValue的当前UID的最大值。当为新的metrics,tagKey和tagValue分配了新的UID后，会更新对应的最大值
* RowKey为”1″的行中，RowKey为UID，Qualifier为”id:metrics”的值”metrics”，Qualifier为”id:tagk”的值为tagKey，Qualifier为id:tagv的值为tagValue
* RowKey为”2″的行中，RowKey为UID，Qualifier为”id:tagv”的值为tagValue，暂不存在UID为”2″的metrics和tagKey
* RowKey为”189.120.205.26″的行中，Qualifer为”name:tagv”的值为UID。表示当”189.120.205.26″为tagValue时，其UID为1
* RowKey为”189.120.205.27″的行中，Qualifer为”name:tagv”的值为UID。表示当”189.120.205.26″为tagValue时，其UID为2
* RowKey为”cpu.hum”的行中，Qualifer为”name:metrics”的值为UID。表示当cpu.hum为metrics时，其UID为1
* RowKey为”host”的行中，Qualifer为”name:tagk”的值为UID。表示当host为tagValue时，其UID为1
---------------------

**由于HBase的存储数据类型是Bytes，所以UID在存储时会被转换为3个字节长度的Bytes数组进行存储**

4. **TSUID**

   对每一个Data Point，metrics、timestamp、tagKey和tagValue都是必要的构成元素。除timestamp外，metrics、tagKey和tagValue的UID就可组成一个TSUID，每一个TSUID关联一个时间序列，如下所示：

> <metrics_UID><tagKey1_UID><**tagValue1_UID**>**[…<tagKeyN_UID><**tagValueN_UID**>**]

上述例子中，就涉及两个TSUID，分别是：

![TSUID](.\images\TSUID-1.png)



## 2.表设计

​	OpenTSDB共涉及两种类型的数据：Metrics数据以及Annotation(注释)数据，在将这些数据存到HBase表中时，针对RowKey, Qualifier以及Value信息都做了特殊设计，从而使得存储更加高效

### 1.Metrics RowKey设计

​	metrics数据的HBase RowKey中包含主要组成部分为：盐值（Salt）、metrics名称、时间戳、tagKey、tagValue等部分。为了统一各个值的长度以及节省空间，对metrics名称、tagKey和tagValue分配了UID信息。所以，在HBase RowKey中实际写入的metrics UID、tagKey UID和tagValue UID。HBase RowKey的数据模型如下图所示：

![RowKey](.\images\RowKey.png)
* SALT：建议开启SALT功能，可以有效提高性能。SALT数据的长度是变长的：如果SALT的值值少于256，那么只用一个字节表示即可；如果需要设置更大的SALT值，也会相应地占用更多的空间。
* Metric ID：metrics名经过编码后，每个Metric ID的长度为三个字节。
* Timestamp：这里是整点小时时间戳。
* tagKey UID & tagValue UID：tagKey和tagValue经过编码后，每个tagKey UID和tagValue UID的长度都为三个字节。tagKey UID和tagValue UID必须成对出现，最少必须存在1对，最多存在8对。

### 2.Metrics Qualifier设计

Qualifier用于保存一个或多个DataPoint中的时间戳、数据类型、数据长度等信息。

由于时间戳中的小时级别的信息已经保存在RowKey中了，所以Qualifier只需要保存一个小时中具体某秒或某毫秒的信息即可，这样可以减少数据占用的空间。

一个小时中的某一秒（少于3600）最多需要2个字节即可表示，而某一毫秒（少于3600000）最多需要4个字节才可以表示。为了节省空间，OpenTSDB没有使用统一的长度，而是对特定的类型采用特性的编码方法。Qualifer的数据模型主要分为如下三种情况：秒、毫秒、秒和毫秒混合。

#### 1.秒类型

当OpenTSDB接收到一个新的DataPoint的时候，如果请求中的时间戳是秒，那么就会插入一个如下模型的数据.判断请求中的时间戳为秒或毫秒的方法是基于时间戳数值的大小，如果时间戳的值的超过无符号整数的最大值（即4个字节的长度），那么该时间戳是毫秒，否则为秒

![秒类型](.\images\Qualifier-Second.png)

* Value长度：Value的实际长度是Qualifier的最后3个bit的值加1，即(qualifier & 0x07) + 1。表示该时间戳对应的值的字节数。所以，值的字节数的范围是1到8个字节。
* Value类型：Value的类型由Qualifier的倒数第4个bit表示，即(qualifier & 0x08)。如果值为1，表示Value的类型为float；如果值为0，表示Value的类型为long。
* 时间戳：时间戳的值由Qualifier的第1到第12个bit表示，即(qualifier & 0xFFF0) >>>4。由于秒级的时间戳最大值不会大于3600，所以qualifer的第1个bit肯定不会是1。

#### 2.毫秒类型

当OpenTSDB接收到一个新的DataPoint的时候，如果请求中的时间戳是毫秒，就会插入一个如下模型的数据。

![毫秒类型](.\images\Qualifier-milisecond.png)

* Value长度：与秒类型相同。
* Value类型：与秒类型相同。
* 时间戳： 时间戳的值由Qualifier的第5到第26个bit表示，即(qualifier & 0x0FFFFFC0) >>>6。
* 标志位：标志位由Qualifier的前4个bit表示。当该Qualifier表示毫秒级数据时，必须全为1，即(qualifier[0] & 0xF0) == 0xF0。
* 第27到28个bit未使用。

#### 3.混合类型

当同一小时的数据发生合并后，就会形成混合类型的Qualifier。合并的方法很简单，就是按照时间戳顺序进行排序后，从小到大依次拼接秒类型和毫秒类型的Qualifier即可。

![混合类型](.\images\Qualifier-mix.png)

* 秒类型和毫秒类型的数量没有限制，并且可以任意组合。
* 不存在相同时间戳的数据，包括秒和毫秒的表示方式。
* 遍历混合类型中的所有DataPoint的方法是：
  * 从左到右，先判断前4个bit是否为0xF
  * 如果是，则当前DataPoint是毫秒型的，读取4个字节形成一个毫秒型的DataPoint
  * 如果否，则当前DataPoint是秒型的，读取2个字节形成一个秒型的DataPoint
  * 以此迭代即可遍历所有的DataPoint

### 3.Metrics Value设计

HBase Value部分用于保存一个或多个DataPoint的具体某个时间戳对应的值。由于在Qualifier中已经保存了DataPoint Value的类型和DataPoint Value的长度，所以无论是秒级还是毫秒级的值，都可以用相同的表示方法，而混合类型就是多个DataPoint Value的拼接。

HBase Value按照长度可以分为如下几种类型：

1. 单字节

   当DataPoint Value为long型，且大于等于-128（Byte.MIN_VALUE），且少于或等于127（Byte.MAX_VALUE）的时候，使用1个字节存储。

2. 两字节

   当DataPoint Value为long型，且大于等于-32768（Short.MIN_VALUE且少于或等于32767（Short.MAX_VALUE）的时候，使用2个字节存储。

3. 四字节

   当DataPoint Value为long型，且大于等于0x80000000（Integer.MIN_VALUE,且少于或等于0x7FFFFFFF（Integer.MAX_VALUE）的时候，使用4个字节存储。

4. 八字节

   当DataPoint Value为long型，且不是上面三种类型的时候，使用8个字节存储。当DataPoint Value为float型的时候，使用8个字节表示。

5. 多字节

   按照时间戳的顺序，把多个Value拼接起来的数据模型如下:

![多字节](.\images\Value-multibytes.png)

- 每个格子表示一个DataPoint Value的值，这个DataPoint Value的长度可能是1或2或4或8个字节。
- DataPoint Value的顺序与Qualifier中时间戳的顺序一一对应。
- 混合标志：如果最后1个字节为0x01，表示存在秒级类型和毫秒级类型混合的情况。

## 3.Annotation数据

Annotation用于描述某一个时间点发生的事件，Annotation的数据为字符串类型，这与数字类型的metrics数据并不同。

>注意:
>
>1. Annotation数据只支持秒级时间戳的数据。
>2. Annotation数据不会合并。

### 1.Annotation RowKey设计

RowKey的数据模型如下图：

![Annotation RowKey](.\images\Annotation-RowKey.png)

* SALT/ Timestamp/Metric UID/ tagKey UID /tagValue UID的意义与metrics RowKey中的意义相同。
* 把[Metric UID/ tagKey UID /tagValue UID]部分统称为TSUID。实际上，读写注释数据的时候，需要指定的是TSUID，而不是像metrics数据中那样分开指定的。

### 2.Annotation Qualifier设计

由于注释数据只支持秒级类型的数据，同时注释类型的数据不支持合并，所以Qualifier的设计相对metrics数据简单一些。Qualifier定义如下：

![Annotation Qualifier](.\images\Annotation-Qualifier.png)

* 与metrics数据的Qualifier相比，注释数据的HBase Qualifer的长度是3个字节。
* 标志位：使用第1个字节表示，而且值必须为0x01。即(qualifier & 0xFF0000)>>>16 == 0x01。
* 时间戳：使用第2到第3个字节表示。即时间戳的值为(qualifier & 0x00FFFF)。

### 3.Annotation Value设计

注释数据中的Value保存的是字符串类型的数据，整个HBase Value部分就是注释数据的值。

## 4.Append模式

当OpenTSDB启动APPEND模式后，每个插入的新DataPoint，都会以HBase的append的方式写入。

> 注意：
>
> 1. 由于使用了HBase的append的接口，每次插入一个新数据，都需要对同一小时的数据都执行一次读取和插入的操作；另外多线程对同一小时的数据进行更新的时候，是不能并发的。这样就大大限制了数据写入的速度了，一般情况下不建议使用这种模式。
> 2. append的数据其实就是合并过的数据了，所以不会参与OpenTSDB的Compaction流程。

### 1.Append模式RowKey设计

Append模式的RowKey设计与普通模式下写入的metrics数据的RowKey是相同的。

### 2.Append模式Qualifier设计

Append模式下，由于同1小时的数据中不存在多个Qualifier，所以只需要使用一个固定的Qualifier即可。

![Append模式Qualifier](.\images\Append-Qualifier.png)

* Append模式的Qualifier使用3个字节表示
* 标志位： 由第1个字节表示，而且值必须为0x05。即(qualifier & 0xFF0000)>>>16 == 0x05
* 固定部分：由第2到第3个字节表示，这部分的值固定为0x0000，因此，Append模式的Qualifier固定为0x050000

### 3.Append模式Value设计

Append模式下， Value部分既要保存时间戳，数值类型和数值长度，也要保存对应的数值。Value的数据结构如下:

![Append模式Value](.\images\Value-Append.png)

* 上图每一个方块表示的Qualifier与Value的定义，与普通写入模式下的定义相同
* 遍历Value中的所有DataPoint的方法是：
  * 从左到右，先判断前4个bit是否为0xF
  * 如果是，则当前DataPoint是毫秒型的读取4个字节形成一个毫秒型的Qualifier，从Qualifier中获得Value的长度，然后再读取对应长度的字节数
  * 如果否，则当前DataPoint是秒型的，读取2个字节形成一个秒型的Qualifier，从Qualifier中获得Value的长度，然后再读取对应长度的字节数；
  * 依此迭代即可遍历所有的DataPoint



## 3.Tree

​	除了元数据，OpenTSDB 2.0还引入了**树**的概念，这是一种将时间序列组织成易于导航的结构的分层方法，可以像计算机上的文件系统一样进行浏览。用户可以使用各种规则集定义多个树，这些规则集将TSMeta对象组织为树结构。然后，用户可以通过HTTP API端点浏览生成的树。有关详细信息，请参见[/ api / tree](http://opentsdb.net/docs/build/html/api_http/tree/index.html)。

**树相关的术语：**

- **分支** - 每个分支是树的一个节点。它包含子分支和叶子列表以及父分支列表。
- **Leaf** - 分支的结尾，代表一个独特的时间序列。叶子将包含可用于生成TSD查询的TSUID值。分支可以并且可能会有多个叶子
- **Root** - 根分支是树的开头，所有分支都从该根伸出。它的深度为0。
- **深度** - 每次将分支添加到另一个分支时，深度会增加
- **严格匹配** - 启用时，时间序列必须与规则集的每个级别中的规则匹配。如果一个或多个级别无法匹配，则时间序列将不会包含在树中。
- **路径** - 层次结构中当前分支上方的每个分支的名称和级别




## 参考资料

[OpenTSDB官方 Http API](http://opentsdb.net/docs/3x/build/html/api_http/index.html#api-endpoints)

[java代码使用Post请求向opentsdb写入数据](https://blog.csdn.net/qq_38417913/article/details/81666962)

[OpenTSDB安装与使用](https://blog.csdn.net/liuxiao723846/article/details/52351919)

[OpenTSDB](https://www.cnblogs.com/shuiyelifang/p/7909594.html)

[解密OpenTSDB的表存储优化](https://yq.aliyun.com/articles/54785)




