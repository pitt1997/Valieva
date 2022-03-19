# Kafka Guide

Kafka 工作常用指南。

## Kafka 常用命令

Kafka常用命令记录。

### Kafka历史数据保存策略配置

进入kafka配置文件，修改 server.properties 文件。（该文件可配置Kafka历史消息保存时间等设置）

```properties
log.retention.hours=1　　　　　　　　　　 # 超过1个小时就清理数据
log.segment.bytes=5000　　　　　　　　　　# 数据量超过5000byte就清理数据
log.cleanup.interval.mins=100　　　　　　# 指定日志每隔多久检查看是否可以被删除，默认1分钟
log.retention.check.interval.ms=300　　 # 文件大小检查的周期时间，是否触发 log.cleanup.policy中设置的策略
```

### 查看kafka版本

```bash
find / -name kafka_\*  | head -1 |grep -o '\kafka[^\n]*'
```

### Topic相关命令

#### 查看kafka中所有的Topic

```bash
./bin/kafka-topics.sh --zookeeper localhost:2181 --list
./bin/kafka-topics.sh --list --bootstrap-server localhost:9092
```

#### 创建Topic

```bash
bin/kafka-topics.sh --create --zookeeper localhost:2181 --topic test_topic --replication-factor 1 --partitions 1 

bin/kafka-topics.sh --create --zookeeper localhost:2181 --topic test_topic --replication-factor 3 --partitions 10 --config cleanup.policy=compact

bin/kafka-topics.sh --create --zookeeper localhost:2181  --topic test_topic --partitions 1   --replication-factor 1 --config max.message.bytes=64000 --config flush.messages=1
```

#### 查看某Topic具体情况

```bash
bin/kafka-topics.sh --zookeeper localhost:2181 --describe --topic test_topic
```

#### 修改topic(分区数、特殊配置如compact属性、数据保留时间等)

```bash
bin/kafka-topics.sh --zookeeper localhost:2181 --alter --partitions 3  --config cleanup.policy=compact --topic test_topic
```

#### 修改Topic

```bash
bin/kafka-configs.sh --alter --zookeeper localhost:2181 --entity-name test_topic --entity-type topics --add-config cleanup.policy=compact
bin/kafka-configs.sh --alter --zookeeper localhost:2181 --entity-name test_topic --entity-type topics --delete-config cleanup.policy
```

### offset相关命令

#### 最大offset

```
bin/kafka-run-class.sh kafka.tools.GetOffsetShell --broker-list localhost:9092 --topic test_topic --time -1
```

#### 最小offset

```
bin/kafka-run-class.sh kafka.tools.GetOffsetShell --broker-list localhost:9092 --topic test_topic --time -2
```

#### 当前offset

```
bin/kafka-run-class.sh kafka.tools.GetOffsetShell --broker-list localhost:9092 --topic test_topic
```

### consumer-group相关命令

#### 查看kafka有那些 group ID 【消费者组】

```
./bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --list
./bin/kafka-consumer-groups.sh --new-consumer --bootstrap-server localhost:9092 --list
```

注意：这里面是没有指定 topic 的，所以查看的所有的 topic 的 消费者 的 group.id 的列表。重名的 group.id 只会显示一次。

#### 查看kafka查看某消费组(consumer_group)具体消费情况(活跃的消费者以及lag情况等等)

```
./bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group vbh_sync --describe
./bin/kafka-consumer-groups.sh --new-consumer --bootstrap-server localhost:9092 --group vbh_sync --describe
```

旧版

```
./bin/kafka-consumer-groups.sh --zookeeper 127.0.0.1:2181 --group test_group --describe
```

### consumer相关命令

#### 消费数据(从latest消费最新)

```
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test_topic
```

旧版

```
bin/kafka-console-consumer.sh --zookeeper localhost:2181 --topic test_topic
```

#### 消费数据(从头开始消费)

```
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test_topic --from-beginning
```

#### 消费数据(最多消费多少条就自动退出消费)

```
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test_topic --max-messages 1
```

#### 消费数据(同时把key打印出来)

```
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test_topic --property print.key=true
```

### producer相关

#### 生产数据

```
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test_topic
```

#### 生产数据(写入带有key的message)

```
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test_topic --property "parse.key=true" --property "key.separator=:"
```

### 监控阻塞消息

```
./bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group vbh_sync
```

### kafka如何从头消费历史数据

消费者要从头开始消费某个topic的全量数据，需要满足2个条件（spring-kafka）：

（1）使用一个全新的"group.id"（就是之前没有被任何消费者使用过）; （2）指定"auto.offset.reset"参数的值为earliest；

```
./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic Topic_VBH --from-beginning --group vbh_sync222 >> /data/4a/a.txt
```

### 项目上大量消息阻塞情况消费处理

```
1、先切换至目录： /data/4a/SyncServer/bin/
2、停止同步组件 执行./server stop
3、切换至目录：/data/4a/KafkaServer/
4、执行命令
	直接执行：./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic Topic_VBH --from-beginning --group vbh_sync
	将消息写入文件：./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic Topic_VBH --from-beginning --group vbh_sync >> /data/4a/a.txt
5、等命令行显示消息刷完之后（消息不再刷新）  ctrl+c退出会话  
6、重启同步组件：至目录/data/4a/SyncServer/bin/   执行./server start 
```

注意：

**1、ls > test.txt    把输出转向到指定的文件，如文件已存在的话也会重新写入，文件原内容不会保留**

**2、ls >> test.txt   是把输出附向到文件的后面，文件原内容会保留下来**

```
切换至目录：/data/4a/KafkaServer/
执行命令

./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic Topic_VBH --from-beginning --group vbh_sync_tmp >> /data/4a/a.txt
```

### 添加Kafka服务host

当时现场网络环境NAT映射后，导致其他应用访问不到平台的kafka，就改成用域名发布

```
vi /etc/hosts
127.0.0.1 qa.kafka.com
::1 4a.kafka.com
```



## Kafka 问题记录

### Broker may not be available 问题记录

[Producer clientId=producer-1] Connection to node -1 could not be established. Broker may not be available.

message.max.bytes：kafka能够接受的最大消息大小，默认是977KB，用户可以根据需要进行设置。

1.进入服务器

2.进入kafka：cd kafka

3.重启kafka：./startup.sh

4.进入zookeeper：cd /home/tmkj/zookeeper/bin

5.重启zookeeper：./zkServer.sh restart

按以上步骤操作集群中的其他服务器，即可重启kafka集群。

```
bin/zookeeper-server-start.sh config/zookeeper.properties
```

然后启动kafka服务

```
bin/kafka-server-start.sh config/server.properties
```

bin/zookeeper-server-start.sh -daemon config/zookeeper.properties

### Kafka 服务启动报错 Address already in use

kafka启动时报错：Socket server failed to bind to 0.0.0.0:9092: Address already in use.

```
JPS
kill -9 Kafka进程
```

A broker is already registered on the path /brokers/ids/0

```
[2021-11-25 10:21:00,179] FATAL [KafkaServer id=0] Fatal error during KafkaServer startup. Prepare to shutdown (kafka.server.KafkaServer)
java.lang.RuntimeException: A broker is already registered on the path /brokers/ids/0. This probably indicates that you either have configured a brokerid that is already in use, or else you have shutdown this broker and restarted it faster than the zookeeper timeout so it appears to be re-registering.
        at kafka.utils.ZkUtils.registerBrokerInZk(ZkUtils.scala:440)
        at kafka.utils.ZkUtils.registerBrokerInZk(ZkUtils.scala:426)
        at kafka.server.KafkaHealthcheck.register(KafkaHealthcheck.scala:73)
        at kafka.server.KafkaHealthcheck.startup(KafkaHealthcheck.scala:53)
        at kafka.server.KafkaServer.startup(KafkaServer.scala:287)
        at kafka.server.KafkaServerStartable.startup(KafkaServerStartable.scala:38)
        at kafka.Kafka$.main(Kafka.scala:92)
        at kafka.Kafka.main(Kafka.scala)
```

```
[root@localhost ~]# cd /data/4a/KafkaServer/
[root@localhost KafkaServer]# ./bin/zookeeper-shell.sh localhost:2181 ls /brokers/ids    
Connecting to localhost:2181

WATCHER::

WatchedEvent state:SyncConnected type:None path:null
[0]
[root@localhost KafkaServer]# 

[root@localhost KafkaServer]# ./bin/zookeeper-shell.sh localhost:2181 delete /brokers/ids/0
Connecting to localhost:2181

WATCHER::

WatchedEvent state:SyncConnected type:None path:null
```

万能重启进行处理

```
1、ps -ef|grep ZooKeeper
kill 全部ZooKeeper进程
2、jps
kill kafka进程
3、service ZooKeeper restart
4、service KafkaServer restart
```



