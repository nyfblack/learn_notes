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





















































