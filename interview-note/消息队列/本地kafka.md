注意：<KAFKA_BROKER> 等于节点地址 例如：“localhosy:9092”

1、启动zookeeper服务（control + c 终止服务）

	（1）进入目录：cd ~/kafka
	（2）执行命令：bin/zookeeper-server-start.sh config/zookeeper.properties
	    或者 bin/zookeeper-server-start.sh -daemon config/zookeeper.properties（以守护的方式启动zookeeper服务，不占用终端窗口）

2、启动Kafka服务（control + c 终止服务）

	（1）进入目录：cd ~/kafka
	（2）执行命令：bin/kafka-server-start.sh config/server.properties
		或者 bin/kafka-server-start.sh -daemon config/server.properties（以守护的方式启动kafka服务，不占用终端窗口）

3、创建主题“test：

	（1）执行命令：bin/kafka-topics.sh --create --topic test --bootstrap-server localhost:9092 --partitions 1 --replication-factor 1

4、查看已经创建主题：

	（1）进入目录：cd ~/kafka
	（2）执行命令：bin/kafka-topics.sh --list --bootstrap-server localhost:9092

5、启动生产者producer，向主题“test”发送消息

	（1）执行命令：bin/kafka-console-producer.sh --topic test --bootstrap-server localhost:9092
	 在生产者窗口中输入消息（每输入一行消息并回车，消息就会被发送到 Kafka），应该能在消费者窗口中看到发送的消息。

6、启动消费者consumer

	（1）执行命令：bin/kafka-console-consumer.sh --topic test --bootstrap-server localhost:9092 --from-beginning

7、关闭所有kafka节点服务

	（1）进入目录：cd ~/kafka
	（2）执行命令：bin/kafka-server-stop.sh

8、查看所有broker节点信息

	（1）进入目录：cd ~/kafka
	（2）执行命令：bin/kafka-broker-api-versions.sh --bootstrap-server <KAFKA_BROKER>

9、查看所有分区

    （1）bin/kafka-topics.sh --bootstrap-server <KAFKA_BROKER> --list

10、查看所有消费者组信息

	（1）bin/kafka-consumer-groups.sh --bootstrap-server <KAFKA_BROKER> --list

11、查看某一个主题的分区情况

	（1）sh kafka-topics.sh --describe --topic <topic_name> --bootstrap-server  <KAFKA_BROKER>

12、查看某一个分区的信息

	（1）sh kafka-console-consumer.sh --bootstrap-server <KAFKA_BROKER> --topic <TOPIC> --partition <PARTITION_ID> --offset <OFFSET> --max-messages <NUM_MESSAGES>
		
	注：
	<KAFKA_BROKER>：Kafka 的一个或多个 broker 地址，例如 `localhost:9092`。
	<TOPIC>：要查看的 Kafka 主题。
	<PARTITION_ID>：你要查看的分区 ID（例如 `0`、`1`、`2` 等）。
	<OFFSET>：希望从哪个偏移量开始消费消息。可以使用 `latest`、`earliest` 来指定。
		    earliest：从最早的消息开始消费。
		    latest：从最新的消息开始消费。
	<NUM_MESSAGES>：消费的最大消息数量。可以指定一个数字，表示最多消费多少条消息。


常用命令：
![[Pasted image 20241125175741.png]]


本地ZooKeeper

1、连接ZooKeeper 命令行终端

	执行命令 sh zookeeper-shell.sh -server localhost:2181

	(1) 查看所有broker
		ls /brokers/ids
	
	(2) 查看特定broker信息
		get /brokers/ids/0
 
	(3) 查看 Kafka 主题的元数据
		ls /brokers/topics
	
	(4) 查看消费者组信息
		ls /consumers

	(5) 查看消费者组偏移量
		ls /consumers/<group_name>/offsets

	(6) 退出
		quit