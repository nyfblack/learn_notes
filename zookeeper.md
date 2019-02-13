# 1.zookeeper概述

<http://zookeeper.apache.org/> 

zookeeper是分布式系统中的协调系统，可提供的服务主要有：配置服务、名字服务、分布式同步、组服务等 

1. zookeeper的选举机制：

   每台服务器一上线就给自己投一票；如果不能满足得到票数超过一半，就把票投给id大的节点

   e.g.有5台服务器，启动时的选举机制；

   - 1上线后给自己投票；1的票数为1不超过一半；
   - 2上线后给自己投票同时1也把票投给2；2的票数为2不超过一半；
   - 3上线后给自己投票同时1和2都把票投给3；3的票数为3，超过一半，立即升级为leader；以后上线的服务都为flower

2. zookeeper的节点类型

   短暂的znode在客户端与服务器端断开（无论是明确的断开还是故障断开）连接时，该znode都会被删除；

   持久的znode在客户端与服务器端断开连接时，该znode不会被删除：

   ​	简单的持久化节点；

   ​	带编号的持久化节点；可以通过对编号排序，知道集群中的服务器的上线顺序； 

   



# 2.zookeeper的安装启动

## 1.本地安装

- 安装
  - 安装jdk ;
  - 下载<http://zookeeper.apache.org/>  zookeeper上传至服务器并解压
  - 修改配置：conf目录下创建配置文件zoo.cfg，配置信息可参考统计目录下的zoo_sample.cfg文件：

```properties
#tickTime:指定了ZooKeeper的基本时间单位（以毫秒为单位）
tickTime=2000	
#initLimit:指定了启动zookeeper时，zookeeper实例中的随从实例同步到领导实例的初始化连接时间限制，超出时间限制则连接失败（以tickTime为时间单位）
initLimit=10	
#syncLimit:指定了zookeeper正常运行时，主从节点之间同步数据的时间限制，若超过这个时间限制，那么随从实例将会被丢弃（以tickTime为时间单位）；
syncLimit=5	
#dataDir:zookeeper存放数据的目录
dataDir=/opt/zookeeper-data/	
#clientPort:用于连接客户端的端口
clientPort=2181		
```

- 启动一个本地的zookeeper服务端

  ```shell
  $ bin/zkServer.sh start
  #检查ZooKeeper是否正在运行;
  $ bin/zkServer.sh status
  #关闭zookeeper服务
  $ bin/zkServer.sh stop
  ```

- 启动一个客户端

  ```shell
  $ bin/zkCli.sh		#启动客户端
  $ bin/zkCli.sh -server localhost  #启动客户端并连接到服务上
  ] ls /     #查看根节点下有哪些子节点，可以双击Tab键查看更多命令
  ```

- Java客户端

  ​	可创建org.apache.zookeeper.ZooKeeper对象来作为zk的客户端，注意，java api里创建zk客户端是异步的，为防止在客户端还未完成创建就被使用的情况，这里可以使用同步计时器，确保zk对象创建完成再被使用。

  ​	使用同步计数器CountDownLatch，在connect方法中创建执行了zk = new ZooKeeper(hosts, SESSION_TIMEOUT, this);之后，下边接着调用了CountDownLatch对象的await方法阻塞，因为这是zk客户端不一定已经完成了与服务端的连接，在客户端连接到服务端时会触发观察者调用process()方法，我们在方法里边判断一下触发事件的类型，完成连接后计数器减一，connect方法中解除阻塞。



## 2.集群安装

​	集群的每个服务器的安装都同本地安装；每台zookeeper启动，也同本地安装的启动；

- 给集群中的每个zookeeper一个唯一的id；

  ```shell
  cd dataDir   #进入配置文件配置的dataDir（存放数据）目录
  vi myid   #在该文件中写入唯一id即可
  ```

- 修改配置文件；添加集群配置

  ```properties
  tickTime=2000	
  initLimit=10	
  syncLimit=5	
  dataDir=/opt/zookeeper-data/	
  clientPort=2181	
  ####################cluster################
  server.2=192.168.155.10:2888:3888
  server.3=192.168.155.11:2888:3888
  server.4=192.168.155.12:2888:3888
  ```

  ```properties
  server.A=B:C:D
  #A:在上一步设置的标识zookeeper的唯一id，表示第几号服务器
  #B:服务器的ip地址
  #C:服务器与集群中Leader服务器交换信息的端口
  #D:执行选举的时候，服务器之间相互通信的端口
  ```

  

# 3.客户端命令行操作

```shell
help	#显示所有操作命令
ls path [watch]		#使用ls命令来查看当前znode中所包含的内容
	$ ls /
	$ ls /sanguo	#注意： ls /sanguo/  这种用法是错误的
	$ ls /sanguo watch  #注册监听节点子节点数量的变化；
ls2 path [watch]	#查看当前节点详细数据并能看到更新次数等数据
	$ ls2 /
create	#普通创建：-s 含有序列；-e 零时（重启或超时消失）
	$ create /sanguo "dongzhuo"	#创建节点的同时，需要写入数据
	$ create /sanguo/shuguo "liubei"
	$ create -e /sanguo/wuguo "sunquan"  #创建短暂节点（重启消失）
	$ create -s /sanguo/weiguo "caocao"  #创建带序号节点（编号是自动递增的）
get path [watch]	#获得节点值
	$ get /sanguo/shuguo  #就可以拿出 liubei 来
	$ get /sanguo watch   #监听节点 /sanguo 数据的变化；注册一次，只能监听一次变化；
set		#设置节点的具体值
	$ set /sanguo/shuguo "liuchan"   #变更节点的值
stat	#查看节点的状态
delet	#删除节点
	$ delete /sanguo/shuguo   #如果该节点有子节点，删除失败
rmr		#递归删除节点
	$ rmr /sanguo  #删除节点包括子节点
```

- stat结构体：

  - dataLength-znode的数据长度
  - numChildren-znode的子节点数量

- zookeeper监听器

  ```shell
  $ ls /sanguo watch  #注册监听节点子节点数量的变化；
  $ get /sanguo watch   #监听节点 /sanguo 数据的变化；注册一次，只能监听一次变化；
  ```



# 4.java API操作

## 1.ZooKeeper中节点的权限：

​Zookeeper的ACL（Access Control List 访问控制列表），分为三个维度：scheme、id、permission，通常表示为：scheme : id : permission，schema代表授权策略，id代表用户，permission代表权限。

1. scheme

   scheme即采取的授权策略，每种授权策略对应不同的权限校验方式 ；

   - world：默认方式，相当于全世界都能访问
   - auth：不使用任何id，表示任何经过身份验证的用户。
   - digest：即用户名:密码这种方式认证，这也是业务系统中最常用的,使用username:password字符串生成MD5哈希，然后将其用作ACL的ID标识。通过以明文形式发送 例如：wangsaichao:123456 来完成身份验证。在ACL中使用时，表达式将是wangsaichao:G2RdrM8e0u0f1vNCj/TI99ebRMw=。
   - ip：使用Ip地址认证

2. id

   id是验证模式，不同的scheme，id的值也不一样。

   - scheme为digest时，id的值为：username:BASE64(SHA1(password))。
   - scheme为ip时，id的值为客户端的ip地址。
   - scheme为world时，id的值为anyone。
   - scheme为auth时,id为 username:password。

3. permission

   - `CREATE(c)`：创建子节点的权限
   - `DELETE(d)`：删除节点的权限
   - `READ(r)`：读取节点数据的权限
   - `WRITE(w)`：修改节点数据的权限
   - `ADMIN(a)`：设置子节点权限的权限




## 2.zookeeper java api

pom.xml

```xml
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>3.4.10</version>
</dependency>
```

ZookeeperDemo.java

```java
public class ZookeeperDemo {
    //连接的zookeeper集群；注意服务器之间用逗号隔开，且没有空格
    private String connectString="192.168.88.81:2181,192.168.88.82:2181,192.168.88.83:2181";
    //超时时间
    private int sessionTimeout=2000;
    private ZooKeeper zooKeeper;
    @Before
    public void init() throws IOException {
        zooKeeper = new ZooKeeper(connectString, sessionTimeout, new Watcher() {
            @Override
            public void process(WatchedEvent watchedEvent) {
                //观察者的回调函数
            }
        });
    }

    /**
     * 节点类型：
     *      PERSISTENT 持久节点
     *      PERSISTENT_SEQUENTIAL 持久带序号节点
     *      EPHEMERAL  临时节点
     *      EPHEMERAL_SEQUENTIAL  临时带序号节点
     */
    @Test
    public void createNode() throws KeeperException, InterruptedException {
        //创建子节点
        String path = zooKeeper.create("/sanguo/shuguo","liubei".getBytes(),
                ZooDefs.Ids.OPEN_ACL_UNSAFE,CreateMode.EPHEMERAL);
        System.out.println(path);
    }

    @Test
    public void setData() throws KeeperException, InterruptedException {
        //修改节点的值；更新数据是必须要提供znode的版本号（也可以使用-1强制更新），
        // 如果版本号不匹配，则更新会失败。
        Stat stat = zooKeeper.setData("/sanguo/shuguo","liuchan".getBytes(),-1);
        System.out.println(stat);
    }

    @Test
    public void getAndWatch() throws KeeperException, InterruptedException {
        //获取子节点：第二个参数：false表示不是观察者，true表示是观察者；
        List<String> children = zooKeeper.getChildren("/sanguo",false);
        for (String child:children) {
            System.out.println("child node:"+child);
            //获取节点数据
            byte[] date = zooKeeper.getData("/sanguo/"+child,false,null);
            System.out.println("child date:"+new String(date));
        }
    }

    @Test
    public void existNode() throws KeeperException, InterruptedException {
        //判断节点是否存在
        Stat stat = zooKeeper.exists("/sanguo/shuguo",false);
        System.out.println(stat==null?"not exist":"exist");
    }

    @Test
    public void deleteNode() throws KeeperException, InterruptedException {
        //删除节点
        zooKeeper.delete("/sanguo/shuguo",-1);
    }

    @Test
    public void registWatcher(){
        //注册监听器；指定默认监听器，覆盖在创建链接的时候指定的监听器
        zooKeeper.register(new Watcher() {
            @Override
            public void process(WatchedEvent watchedEvent) {
                //回调函数
            }
        });
    }
}
```



