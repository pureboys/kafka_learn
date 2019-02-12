### kafka 使用前配置
1. java 1.8+
2. kafka （使用版本2.1）  下载地址: https://mirror.tuna.tsinghua.edu.cn/apache/kafka/
3. zookeeper （使用版本3.4.12） 下载地址: https://mirror.tuna.tsinghua.edu.cn/apache/zookeeper/

### 单机：kafka 使用 (使用kafka自带的zookeeper)
1. 首先要开启zookeeper, 进入到kafka目录中，执行 ./bin/zookeeper-server-start.sh config/zookeeper.properties
2. netstat -tnpl 查看是否监听 2181 端口
3. 开启kafka  ./bin/kafka-server-start.sh config/server.properties
4. netstat -tpln 查看是否监听 9092 端口
5. 创建一个 topic: ./bin/kafka-topics.sh --zookeeper localhost:2181 --create --topic test1 --partitions 3 --replication-factor 1
6. 查看 topic: ./bin/kafka-topics.sh --zookeeper localhost:2181 --describe  --topic test1
7. 创建一个消费者: ./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092  --topic test1
8. 创建一个生产者: ./bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test1  然后输入信息，查看消费者返回。
9. 演示结束后 rm -rf /tmp/* 防止数据干扰 

### 单机：使用自己的zookeeper 启动
1. 启动服务: ./bin/zkServer.sh start conf/zoo_sample.cfg 
2. 查看服务状态: ./bin/zkServer.sh status conf/zoo_sample.cfg
3. ss -tpnl 查看2181端口
4. 开启kafka  ./bin/kafka-server-start.sh config/server.properties
5. netstat -tpln 查看是否监听 9092 端口
6. 创建一个 topic: ./bin/kafka-topics.sh --zookeeper localhost:2181 --create --topic test1 --partitions 3 --replication-factor 1
7. 创建第二个 topoc: ./bin/kafka-topics.sh --zookeeper localhost:2181 --create --topic test2 --partitions 3 --replication-factor 1
8. 查看所有topic: ./bin/kafka-topics.sh --zookeeper localhost:2181 --list 
9. 创建一个消费者: ./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092  --topic test1
10. 创建一个生产者: ./bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test1  然后输入信息，查看消费者返回。

### 集群：配置zookeeper和kafka
1.  配置 zookeeper 集群: 在zookeeper的配置文件conf/zoo_sample.cfg中, 添加集群节点
```
server.1=kafka1:2888:3888
server.2=kafka2:2888:3888
server.3=kafka3:2888:3888
```
并且在 dataDir=/tmp/zookeeper 的每个节点中加入文件分别对应每个节点
```
echo "1" > /tmp/zookeeper/myid
echo "2" > /tmp/zookeeper/myid
echo "3" > /tmp/zookeeper/myid
```

2.  各个节点启动 zookeeper: ./bin/zkServer.sh start conf/zoo_sample.cfg 
3.  查看服务状态: ./bin/zkServer.sh status conf/zoo_sample.cfg
4.  ss -tpnl 查看2181端口
5.  修改 kafka 配置文件中的 server.properties, 
```
broker.id=0
listeners=PLAINTEXT://kafka1:9092
zookeeper.connect=kafka1:2181,kafka2:2181,kafka3:2181
```
尤其是 broker.id 分别改为0,1,2 和 zookeeper.connect 改为 zookeeper 连接地址
6.  开启kafka  ./bin/kafka-server-start.sh config/server.properties
7.  netstat -tpln 查看是否监听 9092 端口
8.  ./bin/kafka-topics.sh --create --topic first --zookeeper kafka1:2181  --partitions 2  --replication-factor 2 创建一个topic
9.  查看 topic 详情状态: ./bin/kafka-topics.sh --describe  --topic first --zookeeper kafka1:2181
10. 查看所有 topic : ./bin/kafka-topics.sh --list --zookeeper kafka1:2181
11. 开启一个生产者: ./bin/kafka-console-producer.sh --broker-list kafka1:9092 --topic first 等待发送消息
12. 开启一个消费者:  ./bin/kafka-console-consumer.sh --bootstrap-server kafka1:9092 --topic first 等待接收消息 --from-beginning 从头开始消费 
13. 删除topic: ./bin/kafka-topics.sh --delete --topic my-replicated-topic   --zookeeper kafka1:2181

### kafka消费者组
1. 在同一个消费者组里的成员不能消费相同topic的分区（patitions）, 例如： 当topic只开启了一个patitions时， 同时开启两个消费者，而且这两个消费者在同一个组里的时候，首先开启 kafka-console-consumer.sh 时会报警告说没有多余的分区。 当生成者发送消息时，只有其中的一个消费者能够收到消息。
2. config/consumer.properties 中有个 `group.id=` 是消费者组
3. 在开启消费者时使用配置 ./bin/kafka-console-consumer.sh --bootstrap-server kafka1:9092 --topic first --consumer.config config/consumer.properties
  
