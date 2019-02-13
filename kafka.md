# 1.kafka简介

## 1.主要功能

​	根据官网的介绍，ApacheKafka®是一个分布式流媒体平台，它主要有3种功能：

- 发布和订阅消息流，这个功能类似于消息队列，这也是kafka归类为消息队列框架的原因
- 以容错的方式记录消息流，kafka以文件的方式来存储消息流
- 可以再消息发布的时候进行处理

## 2.使用场景

- 在系统或应用程序之间构建可靠的用于传输实时数据的管道，消息队列功能
- 构建实时的流数据处理程序来变换或处理数据流，数据处理功能

## 3.工作流程

Kafka目前主要作为一个分布式的发布订阅式的消息系统使用 

1. **消息传输流程**

- **Producer**即生产者，向Kafka集群发送消息，在发送消息之前，会对消息进行分类，即Topic。
- **Topic**即主题，通过对消息指定主题可以将消息分类，消费者可以只关注自己需要的Topic中的消息
- **Consumer**即消费者，消费者通过与kafka集群建立长连接的方式，不断地从集群中拉取消息，然后可以对这些消息进行处理。同一个组的消费者不能同时消费同一个分区的数据；
- 同一个Topic下的消费者和生产者的数量并不是对应的。



2. **kafka服务器消息存储策略**

		创建一个topic时，同时可以指定分区数目，分区数越多，其吞吐量也越大，但是需要的资源也越多，同时也会导致更高的不可用性，kafka在接收到生产者发送的消息之后，会根据均衡策略将消息存储到不同的分区中。在每个分区中，消息以顺序存储，最晚接收的的消息会最后被消费。 



3. **与生产者的交互**

   生产者在向kafka集群发送消息；

- 可以通过指定分区来发送到指定的分区中
- 也可以通过指定均衡策略来将消息发送到不同的分区中
- 如果不指定，就会采用默认的随机均衡策略，将消息随机的存储到不同的分区中



4. **与消费者的交互**

   在消费者消费消息时，使用offset来记录当前消费的位置；每个消费者维护一个自己的offset；

- 在kafka的设计中，可以有多个不同的group来同时消费同一个topic下的消息，他们的的消费的记录位置offset各不相同，不互相干扰。
- 对于一个group而言，消费者的数量不应该多于分区的数量，因为在一个group中，每个分区至多只能绑定到一个消费者上，即一个消费者可以消费多个分区，一个分区只能给一个消费者消费；因此，若一个group中的消费者数量大于分区数量的话，多余的消费者将不会收到任何消息。 



# 2.kafka命令行操作

## 1.kafka的配置

在kafka解压目录下下有一个config的文件夹，里面放置配置文件：

- consumer.properites 消费者配置，这个配置文件用于配置消费者

  ```properties
  #consumer group id
  group.id=test1
  ```

- producer.properties 生产者配置，这个配置文件用于配置生产者

- server.properties kafka服务器的配置，此配置文件用来配置kafka服务器，目前仅介绍几个最基础的配置

  server.properties：如下的配置需要注意，其余配置使用默认即可；

```properties
###########################server basics#################################
#broker.id 申明当前kafka服务器在集群中的唯一ID，需配置为integer;
broker.id=0
#是否可以删除topic；true：可以删除；false(默认)：不能删除
delete.topic.enable=true
###########################log basics#################################
#在kafka目录中创建一个logs文件夹，在此文件夹中存放日志和数据
log.dirs=kafkaDir/logs
#kafka存放数据保存的时间 168h=7day
log.retention.hours=168
#kafka存放数据的大小 1073741824=1G
log.segment.bytes=1073741824
###########################zookeeper#################################
#申明kafka所连接的zookeeper集群的地址，需配置为zookeeper的地址（也可使用内置的 zookeeper）
zookeeper.connect=hadoop102:2181,hadoop103:2181,hadoop104:2181

###########################socket server settings#################################
#申明此kafka服务器需要监听的端口号，如果是在本机上跑虚拟机运行可以不用配置本项，默认会使用localhost的地址，如果是在远程服务器上运行则必须配置；确保服务器的9092端口能够访问；
listeners=PLAINTEXT:// 192.168.180.128:9092
```



## 2.kafka的启动

```shell
cd kafkaDir
#启动zookeeper；（使用的是集成的zookeeper）
bin/zookeeper-server-start.sh config/zookeeper.properties
#启动kafka；
bin/kafka-server-start.sh config/server.properties
#关闭kafka
bin/kafka-server-stop.sh
```


## 3.消息的创建

- topic


```shell
cd kafkaDir
#创建一个名为test的topic；
#replication-factor:副本数为2;即一个leader，一个follower；副本数不能大于集群中的服务器数；
#partitions分2个区；
bin/kafka-topics.sh --create --zookeeper localhost:2181 \
--replication-factor 2 --partitions 2 --topic test
#查看已经创建的topic
bin/kafka-topics.sh --list --zookeeper localhost:2181
#删除topic
bin/kafka-topics.sh --delete --zookeeper localhost:2181 --topic test
#查看topic的详情
bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic test
#######################################################################
Topic：test		PartitionCount:2	 ReplicationFactor:2	Configs:
	Topic:test 	 Partition:0	  Leader:0    Replicas:0,2    Isr:0,2
	Topic:test	 Partition:1	  Leader:1    Replicas:1,0    Isr:1,0
Topic：topic名
Partition：第几号分区
Leader：该分区的leader在哪个服务器上
Replicas：该分区的副本储存在几号服务器上（除了leader就是follower）
Isr：选择leader节点的顺序，如果leader挂了，马上下一个就成为leader
```

- consumer 控制台命令（测试环境用）

```shell
cd kafkaDir
#创建一个消费者;连接zookeeper
#--from-beginning：从开始进行消费；如果不加，默认为消费最新消息
#把offset维护在zookeeper中
bin/kafka-console-consumer.sh --zookeeper localhost:2181 --topic test --from-beginning
#kafka版本高的时候；把offset维护在kafka本地（一个topic中）；
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning
################################################################################################
#启动的时候指定配置文件，配置文件中的配置会生效
bin/kafka-console-consumer.sh --zookeeper localhost:2181 --topic test --from-beginning \
--consumer.config config/consumer.properties
```

- producer 控制台命令（测试环境用）

```shell
cd kafkaDir
#创建一个生产者；在执行完毕后会进入的编辑器页面;连接kafka
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test
#如果向没有创建的topic中发送数据，系统会默认创建这个topic存储数据；根据properties中的配置创建；
```



# 3.kafka java api

pom.xml

```xml
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>0.11.0.2</version>
</dependency>
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka_2.12</artifactId>
    <version>0.11.0.2</version>
</dependency>
```

## 1.topic

```java
public static void main(String[] args) {
    //创建topic
    Properties props = new Properties();
    props.put("bootstrap.servers", "localhost:9092");
    AdminClient adminClient = AdminClient.create(props);
    ArrayList<NewTopic> topics = new ArrayList<NewTopic>();
    //创建了一个名为“topic-test”，分区数为1，副本数为1的Topic
    NewTopic newTopic = new NewTopic("topic-test", 1, (short) 1);
    topics.add(newTopic);
    CreateTopicsResult result = adminClient.createTopics(topics);
    try {
        result.all().get();
    } catch (InterruptedException e) {
        e.printStackTrace();
    } catch (ExecutionException e) {
        e.printStackTrace();
    }
}
```



## 2.Producer生产者

```java
public static void main(String[] args){
    //所有的配置属性都在ProducerConfig.java源码中有说明
    Properties props = new Properties();
    //kafka集群
    props.put("bootstrap.servers", "localhost:9092");
    /**
     * 应答级别，数据写入的ack机制；
     * 0:只写，不管应答；速度最快，但是安全性最低；
     * 1:只要leader应答即可；
     * all:需要leader和follower都应答；速度最慢，但是安全性最高
     * */
    props.put("acks", "all");
    //重试次数，0：表示发送失败不重试
    props.put("retries", 0);
    //提交发送的数据，或者达到设置的批量大小或者达到设置的延时时间，就把数据发送出去；
    props.put("batch.size", 16384); //批量大小
    props.put("linger.ms", 1);  //延时时间
    //整个producer的缓存数据大小
    props.put("buffer.memory", 33554432);
    //key:value 的序列化类
    props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
    props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
	//关联自定义分区类
    props.put("partitioner.class","com.nyf.TestCustomerPartitioner");
    
    KafkaProducer<String, String> producer = new KafkaProducer<>(props);
    for (int i = 0; i < 10; i++){
        //producer.send(new ProducerRecord<String, String>("topic-test", Integer.toString(i)));
        producer.send(new ProducerRecord<String, String>("topic-test", Integer.toString(i)), 
                      (recordMetadata, e) -> {
                //每次发送成功后的回调函数
            });
    }
    producer.close();
}
```

指定分区：

```java
//1.编写自定义分区的类
//2.在配置信息中添加自定义分区的配置信息
public class TestCustomerPartitioner implements Partitioner {
    private Map<String,?> configMap=null;
    @Override
    public int partition(String topic, Object key, byte[] keyBytes, Object value, 
                         byte[] valueBytes,Cluster cluster) {
        //send之后的数据先进入到此类，才会存储到kafka中去；
        //在此处就可以使用config中的一些配置的值了；
        return 0;
    }
    @Override
    public void close() {
        //可以在此处关闭，过程中的一些资源
    }
    @Override
    public void configure(Map<String, ?> configs) {
        //前边创建consumer时，配置文件就是参数configs
        configMap=configs;
    }
}
```



## 3.Consumer消费者

- 高级api

```java
public static void main(String[] args){
    //配置信息
    Properties props = new Properties();
    //kafka集群信息
    props.put("bootstrap.servers", "localhost:9092");
    //消费者组id
    props.put("group.id", "test");
    //是否自动提交offset；true：自动提交
    props.put("enable.auto.commit", "true");
    //自动提交的延时；读完数据，1s后再提交offset；
    props.put("auto.commit.interval.ms", "1000");
    //key:value 的反序列化类
    props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
    props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");

    final KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
    //订阅消费的topic
    consumer.subscribe(Arrays.asList("topic-test"),new ConsumerRebalanceListener() {
        public void onPartitionsRevoked(Collection<TopicPartition> collection) {
        }
        public void onPartitionsAssigned(Collection<TopicPartition> collection) {
            //将偏移设置到最开始
            consumer.seekToBeginning(collection);
        }
    });
    while (true) {
        //拉取topic中的数据，100ms拉取一次
        ConsumerRecords<String, String> records = consumer.poll(100);
        for (ConsumerRecord<String, String> record : records){
            System.out.printf("offset = %d, key = %s, value = %s%n", record.offset(), 
                              record.key(), record.value());
        }
    }
}
```

- 低级api

  使用低级api读取指定topic的指定partition指定offset的数据；

1. 使用步骤

   - 根据指定的分区从主题元数据中找到主副本
   - 获取分区最新的消费进度
   - 从主副本拉取分区消息
   - 识别主副本的变化，重试

2. 方法描述

   findLeader()	:  客户端向种子节点发送主题元数据，将副本集加入备用节点

   getLastOffset()   :   消费者客户端发送偏移量请求，获取分区最近的偏移量

   run()   :  消费者低级api拉取消息的主要方法

   findNewLeader()   :  当分区的主副本节点发生故障，客户将要找出新的主副本









# 4.spring-kafka

- 添加maven依赖

```xml
<!--kafka start-->
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>0.11.0.1</version>
</dependency>
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-streams</artifactId>
    <version>0.11.0.1</version>
</dependency>
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
    <version>1.3.0.RELEASE</version>
</dependency>
```

- 创建配置类

```java
@Configuration
@EnableKafka
public class KafkaConfig {
	//topic config Topic的配置开始
    @Bean
    public KafkaAdmin admin() {
        Map<String, Object> configs = new HashMap<String, Object>();
        configs.put(AdminClientConfig.BOOTSTRAP_SERVERS_CONFIG,"192.168.180.128:9092");
        return new KafkaAdmin(configs);
    }

    @Bean
    public NewTopic topic1() {
        return new NewTopic("foo", 10, (short) 2);
    }
	//topic的配置结束
    
    //配置生产者Factory及Template
	//producer config start
    @Bean
    public ProducerFactory<Integer, String> producerFactory() {
        return new DefaultKafkaProducerFactory<Integer,String>(producerConfigs());
    }
    
    public Map<String, Object> producerConfigs() {
        Map<String, Object> props = new HashMap<String,Object>();
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "192.168.180.128:9092");
        props.put("acks", "all");
        props.put("retries", 0);
        props.put("batch.size", 16384);
        props.put("linger.ms", 1);
        props.put("buffer.memory", 33554432);
        props.put("key.serializer", "org.apache.kafka.common.serialization.IntegerSerializer");
        props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        return props;
    }
    @Bean
    public KafkaTemplate<Integer, String> kafkaTemplate() {
        return new KafkaTemplate<Integer, String>(producerFactory());
    }
	//producer config end
    
    //配置ConsumerFactory
    //consumer config start
    @Bean
    public ConcurrentKafkaListenerContainerFactory<Integer,String> 
        								kafkaListenerContainerFactory(){
        ConcurrentKafkaListenerContainerFactory<Integer, String> factory = 
            			new ConcurrentKafkaListenerContainerFactory<Integer, String>();
        factory.setConsumerFactory(consumerFactory());
        return factory;
    }
    
    @Bean
    public ConsumerFactory<Integer,String> consumerFactory(){
        return new DefaultKafkaConsumerFactory<Integer, String>(consumerConfigs());
    }

    public Map<String,Object> consumerConfigs(){
        HashMap<String, Object> props = new HashMap<String, Object>();
        props.put("bootstrap.servers", "192.168.180.128:9092");
        props.put("group.id", "test");
        props.put("enable.auto.commit", "true");
        props.put("auto.commit.interval.ms", "1000");
        props.put("key.deserializer", 
                  "org.apache.kafka.common.serialization.IntegerDeserializer");
        props.put("value.deserializer", 
                  "org.apache.kafka.common.serialization.StringDeserializer");
        return props;
    }
	//consumer config end
}
```

- 创建消息生产者

```java
//使用spring-kafka的template发送一条消息 发送多条消息只需要循环多次即可
public static void main(String[] args) throws ExecutionException, InterruptedException {
    AnnotationConfigApplicationContext ctx = 
        new AnnotationConfigApplicationContext(KafkaConfig.class);
    KafkaTemplate<Integer, String> kafkaTemplate = 
        		(KafkaTemplate<Integer, String>) ctx.getBean("kafkaTemplate");
        String data="this is a test message";
        ListenableFuture<SendResult<Integer, String>> send = 
            	kafkaTemplate.send("topic-test", 1, data);
        send.addCallback(new ListenableFutureCallback<SendResult<Integer, String>>() {
            public void onFailure(Throwable throwable) {}
            public void onSuccess(SendResult<Integer, String> integerStringSendResult) {}
        });
}
```

- 创建消费者

```java
//首先创建一个用于消息监听的类，当名为”topic-test”的topic接收到消息之后，这个listen方法就会调用
//同时也需要将这个类作为一个Bean配置到KafkaConfig中
public class SimpleConsumerListener {
    private final static Logger logger = LoggerFactory.getLogger(SimpleConsumerListener.class);
    private final CountDownLatch latch1 = new CountDownLatch(1);

    @KafkaListener(id = "foo", topics = "topic-test")
    public void listen(byte[] records) {
        //do something here
        this.latch1.countDown();
    }
}
```

在配置文件中追加

```java
@Bean
public SimpleConsumerListener simpleConsumerListener(){
    return new SimpleConsumerListener();
}
```

默认spring-kafka会为每一个监听方法创建一个线程来向kafka服务器拉取消息 



































