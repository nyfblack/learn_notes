# 1.大数据

**主要解决：海量数据的储存和海量数据的分析计算问题；**

- 数据存储：

  数据存储单位：bit Byte KB MB GB TB PB EB ZB YB BB NB DB

  换算关系：Byte=8bit   KB=1024Byte   MB=1024KB ......

- 大数据特点(4v)：

  volume(大量)、velocity(高速)、variety(多样)、value(低价值密度)

- 大数据部门业务流程分析：

  产品人员提需求：统计用户数、日活跃用户数、回流用户数...

  数据部门搭建数据平台、分析数据指标

  数据可视化：报表展示、邮件发送、大屏幕展示...

- **大数据部门组织结构**

  **平台组**

  ​	Hadoop、Flume、Kafka、HBase、Spark等框架平台的搭建

  ​	集群性能监控

  ​	集群性能调优

  **数据仓库组**

  ​	ETL工程师-数据清洗

  ​	Hive工程师-数据分析、数据仓库建模

  **数据挖掘组**

  ​	算法工程师

  ​	推荐系统工程师

  ​	用户画像工程师

  报表开发组

  ​	JavaEE工程师



# 2.Hadoop

## 1.Hadoop是什么

1. Hadoop 是一个由 Apache 基金会所开发的分布式系统基础架构；
2. 主要解決：大数据存储、大数据分析；
3. Hadoop广义上讲是Hadoop生态圈；



# 2.Hadoop发展

## 1.Google在大数据方面的三篇论文：

GFS ----> HDFS

Map-Reduce ----> MR

BigTable ----> HBase

## 2.Hadoop的特点（4高）

- 高可靠性 ： Hadoop 按位存储和处理数据的能力值得人们信赖。
- 高扩展性 ： Hadoop 是在可用的计算机集簇间分配数据并完成计算任务的，这些集簇可以方便地扩展到数以干计的节点中。
- 高效性 ： Hadoop能够在节点之间动态地移动数据，并保证各个节点的动态平衡，因此处理速度非常快。
- 高容错性 ： Hadoop能够自动保存数据的多个副本，并且能够自动将失败的任务重新分。

## 3.Hadoop的组成

Hadoop1.x和Hadoop2.x的区别

| Hadoop1.x                                                    | Hadoop2.x                                                    |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| Common(辅助工具)<br />HDFS(数据存储)<br />MapReduce(计算+资源调度) | Common(辅助工具)<br />HDFS(数据存储)<br />MapReduce(计算)<br />Yarn(资源调度) |

- HDFS：
  - NameNode：存储文件的元数据；
  - DataNode：在本地文件系统存储文件块数据；
  - Secondary NameNode：监控HDFS状态的辅助后台程序，每隔一段时间获取HDFS元数据快照；
- Yarn：
  - ResourceManager：整个集群资源的老大
  - NodeManager：某个节点资源的老大
  - ApplicationMaster：某个任务的资源的调度
  - Container：为ApplicationMaster服务，封装了某节点上的多维度资源(cpu/内存/磁盘/网络...)
- MapReduce
  - Map阶段并行处理输入数据
  - Reduce阶段，对map结果进行汇总

## 4.Hadoop生态系统



# 3.Hadoop运行环境的搭建

## 1.虚拟机环境准备

1. 克隆虚拟机
2. 修改克隆虚拟机的静态IP
3. 修改主机名
4. 关闭防火墙

```shell
vim /etc/udev/rules.d/70-persistent-net.rules
vim /etc/sysconfig/network-scripts/ifcfg-eth0
vim /etc/sysconfig/network
vim /etc/hosts
#修改完成后重启
ifconfig  #查看修改是否生效
ping out_ip  #ping一下外边的ip看能不能ping通
#在外边ping一下里边的ip看能不能ping通
```

5. 创建hadoop用户
6. 配置hadoop用户具有root权限

```shell
vim /etc/sudoers
######################配置权限#########################
root	ALL=(ALL)	ALL
hadoop	ALL=(ALL)	ALL
```

5. 在 /opt 下创建文件夹

```shell
sudo mkdir module
sudo mkdir software
chown hadoop:hadoop module software #修改文件的所有者和所在组
```



## 2.安装JDK

1. 上传jdk到software文件中

2. 解压jdk到module目录中

   ```shell
   tar -zxvf /opt/software/jdk.tar.gz -C /opt/module
   ```

3. 复制jdk的路径，配置环境变量

   ```shell
   sudo vim /etc/profile
   ################在文件末尾添加#############################
   ##JAVA_HOME
   export JAVA_HOME=/opt/module/jdk
   export PATH=$PATH:$JAVA_HOME/bin
   #########################################################
   #配置好环境变量后，运行如下命令,使配置生效
   source /etc/profile
   ```

   

## 3.安装Hadoop

1. 上传安装包并解压

   ```shell
   tar -zxvf /opt/software/hadoop.tar.gz -C /opt/module
   ```

2. 配置环境变量

   ```shell
   sudo vim /etc/profile
   ################在文件末尾添加#############################
   ##HADOOP_HOME
   export HADOOP_HOME=/opt/module/hadoop
   export PATH=$PATH:$HADOOP_HOME/bin
   export PATH=$PATH:$HADOOP_HOME/sbin
   #########################################################
   #配置好环境变量后，运行如下命令,使配置生效
   source /etc/profile
   ```



## 4.Hadoop目录结构



# 4.Hadoop运行模式

## 1.本地运行模式

1. 官方Grep案例

```shell
#在hadoop安装目录中运行
$ mkdir input
$ cp etc/hadoop/*.xml input
$ bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar grep input output 'dfs[a-z.]+'
$ cat output/*
```

2. 官方WordCount案例

   **所有的统计汇总需求都是这个模式的**

```shell
#在hadoop安装目录中运行；该案例统计输入文件wc.input中某单词出现的次数
$ mkdir wcinput
$ cd wcinput
$ touch wc.input
$ vi wc.input
###########################################
...#写一些单词
###########################################
$ bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar wordcount wcinput wcoutput
$ cat wcoutput/*
```



## 2.伪分布式模式


## 2.伪分布式模式

### 1.启动HDFS并运行MapReduce

#### 1.分析

- 配置集群
- 启动、测试集群增、删、查
- 运行WordCount案例

#### 2.配置集群：

1. 配置hadoop-env.sh；遇到*env的配置文件，就配置JAVA_HOME

```shell
#添加jdk的路径；
#export JAVA_HOME=${JAVA_HOME}     	#这种配置只能在本地运行，连接集群时，其他节点将不会访问到该属性
export JAVA_HOME=/opt/module/jdk	#这种配置，集群的其他节点也可访问
```

2. 配置core-site.xml

```xml
<configuration>
    <!--HDFS中的NameNode[主节点]的地址-->
	<property>
		<name>fs.defaultFS</name>
		<value>hdfs://1.0.0.5:9000</value>   
	</property>
    
    <!--指定hadoop运行时产生文件的储存目录；修改到hadoop目录下-->
    <property>
		<name>hadoop.tmp.dir</name>
		<value>/opt/module/hadoop/data/tmp</value>   
	</property>
</configuration>
```

3. 配置：hdfs-site.xml

```xml
<configuration>    
	<!--指定HDFS副本数，默认是3个副本-->
    <property>
		<name>dfs.replication</name>
		<value>1</value>   
	</property>
</configuration>
```

#### 3.启动集群

1. 格式化NameNode(第一次启动时格式化，以后不要总格式化)

```shell
$ bin/hdfs namenode -format 
```

2. 启动NameNode

```shell
$ sbin/hadoop-daemon.sh start namenode
```

3. 启动DataNode

```shell
$ sbin/hadoop-daemon.sh start datanode
```



#### 4.测试

- 查看集群

1. 查看集群是否启动成功

```shell
$ jps   #查看namenode和datanode进程是否启动；jps命令在JDK中；
```

2. 登录hadoop提供的web页面

   <ip:50070>

3. 创建路径测试

   ```shell
   $ bin/hdfs dfs -mkdir -p /user/nyf/input
   #创建好后的目录结构，可以在页面上看到
   ```

4. 把本地的文件上传到hdfs上

   ```shell
   $ bin/hdfs dfs -put wcinput/wc.input /user/nyf/input
   #把本地 wcinput/wc.input 文件上传到hdfs上的 /user/nyf/input 目录下；在页面上查看
   ```

5. 执行WordCount案例

   ```shell
   $ bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar wordcount /user/nyf/input /user/nyf/output   #output文件夹依然不能存在
   ###########################运行完成查看结果#########################################
   $ bin/hdfs dfs -cat /user/nyf/output/p*
   ```

6. 运行完后，可以在logs文件夹中查看日志信息



#### 5.关于格式化

为什么不能一直格式化NameNode，格式化NameNode，要注意什么？

```shell
cd data/tmp/dfs/name/current/
cat VERSION   #查看clusterID
```

- 注意：格式化NameNode，会产生新的集群id，导致NameNode和DataNode的集群id不一致，集群找不到以往数据。所以，格式化NameNode时，一定要先删除data数据和log日志，然后再格式化NameNode；

- 要格式化的时候，先用jps看一下进程，如果进程没关掉，格式化后，会自动又生成数据（从别的副本copy过来的）；

### 2.启动YARN并运行MapReduce

#### 1.分析

- 配置集群在YARN上运行MR
- 启动、测试集群的增、删、查
- 在YARN上执行WordCount

#### 2.配置集群

1. 配置yarn-env.sh；配置JAVA_HOME

   ```SHELL
   export JAVA_HOME=/opt/module/jdk
   ```

2. 配置yarn-site.xml

   ```xml
   <configuration>
       <!--reducer获取数据的方式-->
   	<property>
   		<name>yarn.nodemanager.aux-services</name>
   		<value>mapreduce_shuffle</value>   
   	</property>
       
       <!--指定yarn的resourceManager的地址-->
       <property>
   		<name>yarn.resourcemanager.hostname</name>
   		<value>hadoop101</value>   
   	</property>
   </configuration>
   ```

3. 配置mapred-env.sh

   ```shell
   export JAVA_HOME=/opt/module/jdk
   ```

4. 配置：mapred-site.xml （对mapred-site.xml.template重命名为）

   ```shell
   $ mv mapred-site.xml.template mapred-site.xml
   $ vi mapred-site.xml
   #################################################
       <!--指定MR运行在yarn上-->
       <property>
   		<name>mapreduce.framework.name</name>
   		<value>yarn</value>   
   	</property>
   ```

#### 3.启动集群

1. 启动前必须保证NameNode和DataNode已经启动

2. 启动ResourceManager

   ```shell
   $ sbin/yarn-daemon.sh start resourcemanager
   ```

3. 启动NodeManager

   ```shell
   $ sbin/yarn-daemon.sh start nodemanager
   ```

- 启动集群后可以在 :8088 端口看到mr的运行，重新运行WordCount案例，观察web端的显示数据


### 3.配置历史服务器

为了查看程序的历史运行情况，需要配置一下历史服务器。具体步骤如下：

#### 1.配置mapre-site.xml

```xml
<!--增加如下配置-->
<configuration>
    <!--历史服务器端地址-->
	<property>
		<name>mapreduce.jobhistory.address</name>
		<value>hadoop101:10020</value>   
	</property>
    
    <!--历史服务器web端地址-->
    <property>
		<name>mapreduce.jobhistory.webapp.address</name>
		<value>hadoop101:19888</value>   
	</property>
</configuration>
```

#### 2.启动历史服务器

```shell
#配置启动之前，以上HDFS和YARN都是启动的
$ sbin/mr-jobhistory-daemon.sh start historyserver
```

- 查看是否启动成功

```shell
$ jps
```

- 查看jobHistory

  在 :8088 端口的mr的web页面，查看运行的history，就会跳到配置的历史服务器；



### 4.配置日志的聚焦

日志聚集概念：应用运行完成以后，将程序运行日志信息上传到HDFS系统上。

日志聚集功能的好处：可以方便的查看程序运行详情，方便开发调试。

注意：开启日志聚集功能，需要重新启动NodeManager、ResourceManager和HistoryManager

```shell
#关闭以上进程
$ sbin/mr-jobhistory-daemon.sh stop historyserver
$ jps   #查看是否停止
$ sbin/yarn-daemon.sh stop nodemanager
$ jps
$ sbin/yarn-daemon.sh stop resourcemanager
$ jps
```

开启日志聚集功能具体步骤如下：

#### 1.配置yarn-site.xml

```xml
<!--增加如下配置-->
<configuration>
    <!--日志聚集功能使能-->
	<property>
		<name>yarn.log-aggregation-enable</name>
		<value>ture</value>   
	</property>
    
    <!--日志保留时间设置7天-->
    <property>
		<name>yarn.log-aggregation.retain-seconds</name>
		<value>604800</value>   
	</property>
</configuration>
```

- 关闭NodeManager、ResourceManager和HistoryManager

- 启动NodeManager、ResourceManager和HistoryManager

- 删除HDFS上已经存在的输出文件

  ```shell
  $ bin/hdfs dfs -rm -R /user/nyf/output
  ```

- 执行WordCount程序

- 查看日志



### 5.配置文件说明


## 3.完全分布式模式













# 附录：

## 端口

:50070    查看HDFS

:8088    	查看MapReduce运行进程

:10020	历史服务器端地址

:19888	历史服务器web端地址




