一、宏观认知

![[kafka.png]]

![[Pasted image 20241129145155.png]]
	
其主要特性有：解耦、异步、限流、削峰

二、基本构造

	1、组成部分
	
	（1）Producer ：生产者；
	（2）Broker：服务实例，负责消息的持久化、中转等功能；
	（3）Consumer：消费者；
	（4）Zookeeper：负责broker、consumer集群元数据的管理；
	
	2、基本概念
	
	（1）Topic：消息主题
		Kafaka按照topic对消息进行分类，在收发消息时，只需要指定topice即可；
			
	（2）Partition：分区
		为了提升系统的吞吐，一个topic下通常会有多个partition，其中partition分布在不同的broker上，用于存储topic的消息；另外，partition通常会分组，且每组有一个主partition、多个副本partition，且分布在不同的broker中，进而起到容灾的作用；同时，发送的消息会分别分布在各个分区中，例如100条消息，3个分区p1、p2、p3,则p1可能包含20条、p2包含30条、p3包含50条。

	（3）Replication：分区副本
		每个分区可以有多个Replication，由一个Leader和若干个Follower组成。Leader负责接收生产者push的消息和消费者poll消费消息。Follower会实时从自己的Leader中同步数据保持同步。Leader故障时,某个Follower会上位为新的Leader。分区副本的作用是保证高可用。
			
	（4）Segment 分段
		将每个partition分为多个segment，同时也便于消息的维护和清理；
		
	（5）Offset：位移
		用于定位和记录消息在 partition中的位置和消费进度。
		一是用来定位消息。通过指定offset，消费者可以准确地找到分区中的某条消息，或者从某个位置开始消费消息。
		二是用来记录消费进度。消费者在消费完一条消息后，需要提交offset来告诉Kafka broker自己消费到哪里了。这样，如果消费者发生故障或重启，它可以根据保存的 offset 来恢复消费状态。

	（6）ConsumerGroup：消费者组
		同一个消费者组中的多个消费者分摊一个topic中的消息进行消费；不同消费者组中的多个消费者可以共同消费一个topic中的相同消息。每个分区只能由消费者组中的一个消费者进行消费，这样可以保证消息的顺序性和一致性。
		一个主题对应一个消费者组，等价于消息队列；
		一个主题对应多个消费者组，等价于发布订阅；
			
	（7）ISR（In-sync Replicas）：已同步副本
		表示存活且副本都已和Leader同步的的broker集合，是Leader所有replicas副本的子集。如果某个副本节点宕机，该副本就会从ISR集合中剔除。

	（8）ZooKeeper
		Kafka使用ZooKeeper来进行集群管理、协调和元数据存储。Kafka中的Broker、Topic、Consumer都会注册到zookeeper
		
	（9）Producer：生产者
		生产者在向 Kafka 发送消息时，可以指定一个分区键（Partition Key），Kafka 会根据这个键和分区算法来决定消息应该发送到哪个分区。如果没有指定分区键，Kafka会采用轮询或随机的方式来选择分区。生产者也可以自定义分区算法。当消息被写入到分区后，broker会为消息分配一个 offset，并返回给生产者。生产者可以根据返回的 offset 来确认消息是否成功写入，并进行重试或其他处理。
			
	（10）Consumer：消费者
		消费者在消费 Kafka 消息时，需要维护一个当前消费的 offset 值，以及一个已提交的 offset值。当前消费的offset值表示消费者正在消费的消息的位置，已提交的offset值表示消费者已经确认消费过的消息的位置。

	消费者在消费完一条消息后，需要提交offset来更新已提交的offset值。提交offset的方式有两种：自动提交和手动提交。
	- 自动提交：Kafka 提供了一个配置参数 enable.auto.commit，默认为 true，表示开启自动提交功能。自动提交功能会在后台定期（由 auto.commit.interval.ms 参数控制）将当前消费的 offset 值提交给 Kafka broker。
	- 手动提交：如果 enable.auto.commit 设置为 false，则表示关闭自动提交功能，此时消费者需要手动调用 commitSync 或 commitAsync 方法来提交 offset。手动提交功能可以让消费者更灵活地控制何时以及如何提交 offset。

	（11）Controller：控制器
		Controller作为Kafka集群中的核心组件，它的主要作用是在 ZooKeeper 的帮助下管理和协调整个 Kafka 集群。 Controller与Zookeeper进行交互，获取与更新集群中的元数据信息。其他broker并不直接与zookeeper进行通信，而是与 Controller 进行通信并同步Controller中的元数据信息。 Kafka集群中每个节点都可以充当Controller节点，但集群中同时只能有一个Controller节点。
	
		作用：
		（1）主题管理
		（2）分区重分配
		（3）集群成员管理
		（4）元数据服务
			控制器上保存了最全的集群元数据信息，其它所有broker会定期接受控制器发来的元数据更新请求，从而更新其内存中的缓存数据
	
	（12）Coordinator：协调器
		在 Kafka 中，“Coordinator” 是指一种协调者的角色，用于协调不同的操作和功能。不同类型的 Coordinator 在 Kafka 中有不同的作用。
	
		分类：
		（1）触底消费者组协调（Consumer Group Coordination）
			负责管理消费者组中的成员以及分配消息分区，当消费者组内的消费者数量发生变化时，协调器需要重新分配分区，确保每个消费者尽可能均匀地分配到分区上。
			
		（2）生产者协调（Producer Coordination）
			生产者在发送消息时，协调器帮助决定消息的分区，并进行必要的确认。
			
		（3）事务协调（Transactional Coordination）
			事务协调器负责管理事务的生命周期，包括事务的开始、提交和回滚。它确保生产者发送的事务消息在 Kafka 中的一致性。
			
		（4）分区分配与负载均衡（Partition Assignment & Load Balancing）
			负责对消息分区的负载均衡进行协调，确保集群中的消息分区被均匀地分配到不同的 Kafka broker 上。


	注意：
		1、Kafka确保每个分区内的消息是有顺序的，即每个分区中的消息会按照生产者写入的顺序被存储；消费者读取消息时，也是按照分区内的顺序来消费消息。
		2、不同分区内的消息并不保证顺序。例如，消息 A 可能在partition0中，消息B可能在 partition1中。Kafka 不保证消息A会先被消费于消息B，除非消费者明确地按照分区进行消费。
		3、在多broker情况下，某一topic下的分区，会分布在不同的节点上。
		4、如果所有消费者实例都属于同一个消费者组，则实现的是消息队列模型；
		如果所有消费者实例分别属于不同的消费者组，则实现的是发布/订阅模型（即订阅某一主题的各个消费者组，都会获得消息）。

三、存储机制

![[Pasted image 20241118153326.png]]

注意：
	1、 日志文件分段（日志分段文件对应的两个索引文件主要是用来提高查找消息的速度）
	
	生产者生产的消息会不断追加到.log 文件末尾，为防止log文件过大导致数据定位效率低下，Kafka 采取了分片和索引机制，将每个partition 分为多个segment。每个segment都有对应的.index文件和.log文件以及.timeindex文件。这些文件位于kafka的配置文件server.properties中配置项log.dirs所指定目录下的一个文件夹中，该文件夹的命名规则为：topic名称+分区序号。例如test这个topic 设置了三个分区，则会创建对应的文件夹test-0,test-1,test-2。

四、高可靠性

	Kafka 高可靠性的核心是保证消息在传递过程中不丢失，涉及如下核心环节：

	- 消息从生产者可靠地发送至 Broker；例如：网络、本地丢数据。
    
	- 发送到Broker的消息可靠持久化；例如：PageCache缓存落盘、单点崩溃、主从同步跨网络。
    
	- 消费者从Broker消费到消息且最好只消费一次； 例如：跨网络消息传输。

	1、消息从生产者producer发送到broker
	
	（1）发送消息成功后，能够收到broker消息保存成功返回的ACK;
	（2）发送消息失败后，能够捕获超时、失败ACK等异常信号；
	
	注意：
	
	**ACK策略：**
		（1）ack = 0，消息发送即认为成功，不关心有没有写成功，常用于日志进行分析场景；
		（2）ack = 1，leader partition 写入成功以后，才算写入成功，有丢数据的可能；
		（3）ack = -1，ISR列表里面的所有副本都写完以后，这条消息才算写入成功，强可靠性保证；
	
	**消息发送策略：**
	（1）同步发送（sync），默认；

![[sync.jpg]]
		
	  同步发送在一定程度上确保了我们在跨网络向 Broker 传输消息时，消息一定可以可靠地传输到 Broker。因为在同步发送场景我们可以明确感知消息是否发送至 Broker，若因网络抖动、机器宕机等故障导致消息发送失败或结果不明，可通过重试等手段确保消息至少一次（at least once） 发送到 Broker。
		
	（2）异步发送（async），

![[async 4.jpg]]
		
	在主协程中调用异步发送 kafka 消息的时候，其本质是将消息体放进了一个 input 的 channel，只要入 channel 成功，则这个函数直接返回，不会产生任何阻塞。相反，如果入 channel 失败，则会返回错误信息。因此调用 async 写入的时候返回的错误信息是入 channel 的错误信息，至于具体最终消息有没有发送到 kafka 的 broker，我们无法从返回值得知。
	当消息进入 input 的 channel 后，会有另一个**dispatcher 的协程**负责遍历 input，来真正发送消息到特定 Broker 上的主 Partition 上。发送结果通过一个**异步协程**进行监听，循环处理 err channel 和 success channel，出现了 error 就记一个日志。因此异步写入场景时，写 kafka 的错误信息，我们暂时仅能够从这个错误日志来得知具体发生了什么错，并且也不支持我们自建函数进行兜底处理，这一点在 trpc-go 的官方也得到了承认。

	幂等性：
		如果启用了幂等性，则ack默认就是-1
		特征：
			1、单分区幂等性：只能保证单分区上幂等性，无法实现多个分区的幂等性；
			2、单会话幂等性：只能实现单会话上的幂等性，当Producer重启后，这种幂等性保证失效；
		
		实现原理：
		（1）每条消息一个唯一的ID：每次发送消息时，Kafka会给每条消息生成一个唯一的Producer ID和Sequence number（序列号）。Producer ID是在生产者与 Kafka 连接时分配的，Sequence number 是生产者为每个Producer ID分配的递增数字。每个消息都有一个与其对应的唯一标识。
	     （2）重复消息的检测：如果生产者因网络抖动、重试等原因发送了重复的消息（具有相同的 Producer ID和Sequence number），Kafka 会检测到并忽略这些重复的消息，不会将它们写入到 Kafka 的分区中。

	事务：
		特征：
			1、跨分区事务：能够保证将消息原子性地写入到多个分区中；
			2、跨会话的事务恢复：如果一个应用实例挂了，启动的下一个实例依然可以保证上一个事务完成，即（commit 或 abort）
			

	2、发送到broker的消息可靠持久化
		（1）Broker返回 Producer 成功 ack 时，消息是否已经落盘 ？
		（2）Broker宕机是否会导致数据丢失？容灾机制是什么 ？
		（3）Replica副本机制带来的多副本间数据同步一致性问题如何解决 ？

	**Broker异步刷盘机制**：
	
		kafka 为了获得更高吞吐，Broker 接收到消息后只是将数据写入 PageCache 后便认为消息已写入成功，而 PageCache 中的数据通过 linux 的 flusher 程序进行异步刷盘，将数据顺序写到磁盘。由于消息是写入到 pageCache，单机场景，如果还没刷盘 Broker 就宕机了，那么 Producer 产生的这部分数据就可能丢失。为了解决单机故障可能带来的数据丢失问题，Kafka 为分区引入了副本机制。

	**Replica副本机制：**
		Kafka每组分区通常有多个副本，同组分区的不同副本分布在不同的broker上，保存相同的消息（可能存在滞后）。副本之间是“一主多从”的关系，其中 leader 副本负责处理读写请求，follower 副本负责从 leader 拉取消息进行同步。分区的所有副本统称为 AR（Assigned Replicas），其中所有与 leader 副本保持一定同步的副本（包括 leader 副本在内）组成 ISR（In-Sync Replicas），与 leader 同步滞后过多的副本组成 OSR（Out-of-Sync Replicas），由此可见，AR=ISR+OSR。

	小结：Broker 接收到消息后只是将数据写入 PageCache 后便认为消息已写入成功，但是，通过副本机制并结合 ACK 策略可以大概率规避单机宕机带来的数据丢失问题，并通过 HW、副本同步机制、 Leader Epoch 等多种措施解决了多副本间数据同步一致性问题，最终实现了 Broker 数据的可靠持久化。


	3、消费者从broker消费的消息且最好只消费一次
		Consumer 在消费消息的过程中需要向 Kafka 汇报自己的位移数据，只有当 Consumer 向 Kafka 汇报了消息位移，该条消息才会被 Broker 认为已经被消费。因此，Consumer 端消息的可靠性主要和 offset 提交方式有关，Kafka 消费端提供了两种消息提交方式：

![[Pasted image 20241125171105.png]]


五、消费者分区策略
![[Pasted image 20241125180638.png]]
（1）RoundRobin分配策略：

		RoundRobin 分配策略将分区平均地、轮流地分配给消费者。Kafka 会将所有分区按顺序分配给每个消费者，保证每个消费者尽可能平等地消费数据。

	工作原理：
	- Kafka 会按顺序将每个分区轮流分配给消费者。如果消费者数量小于分区数量，那么一些消费者可能会分配到多个分区。
	- 比如，假设有 4 个分区和 3 个消费者，RoundRobin 策略会将分区 0 分配给消费者 1，分区 1 分配给消费者 2，分区 2 分配给消费者 3，分区 3 分配给消费者 1，依此类推。

（2）Range分配策略（默认）：
		
		Range分配策略是 Kafka 默认的分区分配策略，它将 topic 中的分区范围分配给每个消费者。在该策略下，Kafka 会根据消费者的数量和分区的数量将分区进行范围划分，尽量让每个消费者获取连续的分区。
	工作原理：
	- Kafka 会按顺序为每个消费者分配一段范围的分区。
	- 比如，假设有 4 个分区和 3 个消费者，Range 策略会将分区 0 和 1 分配给消费者 1，分区 2 分配给消费者 2，分区 3 分配给消费者3。
	
（3）Sticky分配策略：

		Sticky旨在尽量保持分配的一致性，同时在消费者增加或减少时，尽可能避免重新分配所有的分区。
	工作原理：
	- Sticky 策略尽量保持每个消费者原先所拥有的分区，只有在某个消费者失效或新增消费者时，才会调整分配。
	- 该策略会优先确保尽量少的消费者重新获得以前的分区，如果必须重新分配，只有未被分配的分区才会被考虑。

六、AR、ISR、OSR、HW、LEO概念介绍

	一个Partition中所有副本称为AR (Assigned Replicas) ，所有与 leader 副本保持一定程度同步的副本(包括 leader 副本在内)组成 ISR (In-Sync Replicas)。我们上文提到，follower 副本只负责消息的同步，很多时候 follower 副本中的消息相对 leader 副本而言会有一定的滞后，而及时与leader副本保持数据一致的就可以成为ISR成员。与 leader 副本同步滞后过多的副本(不包括 leader 副本)组成OSR (Out-of-Sync Replicas)，由此可见，AR =ISR+OSR。在正常情况下，所有的 follower 副本都应该与 leader 副本保持一定程度的同步，即 AR=ISR，OSR 集合为空。

	leader副本会监听所有follower副本，当其与leader副本数据一致时会将其加入ISR成员，当与leader副本相差太多或宕机时会将其踢出ISR，也会在其追上leader副本后重新加入ISR。

	当leader副本宕机或不可用时，只有ISR成员才能有机会被选择为新的leader副本，这样就能确保新的leader与已经宕机的leader数据一致，而如果选择OSR中的副本作为leader时会造成部分未同步的数据丢失。

![[Pasted image 20241126180911.png]]

	上图情况中，P1副本首先当选了leader，且只有P2副本同步了P1的数据，offset都为110，那么此时的ISR只有P1与P2，OSR有P3和P4。当P3同步数据到110后，也会被leader加入到ISR中，若此时leader宕机，则会从ISR中选出一个新的leader，并将P0踢出ISR中。

注意：leader副本是如何感知到其它副本是否与自己的数据一致的 ？
	
	答：HW与LEO机制

	LEO 是 Log End Offset 的缩写，它标识当前日志文件中下一条待写入消息的 offset，LEO 的大小相当于当前日志分区中最后一条消息的 offset 值加 1。分区 ISR 集合中的**每个副本都会维护自身的 LEO，而 ISR 集合中最小的 LEO 即为分区的 HW**，HW 是 High Watermark 的缩写，俗称高水位，它标识了一个特定的消息偏移量(offset)，消费者只能拉取到这个 offset 之前的消息。
	
![[Pasted image 20241126181342.png]]

	上图中，因为所有副本消息都是一致的，所以所有LEO都是3，HW也为3，当有新的消息产生时，即leader副本新插入了3/4两条消息，此时leader的LEO为5，两个follower的此时未同步消息，所以LEO仍未3，HW选择最小的LEO是3.

	当follower1同步完成leader的数据后，LEO未5，但follower2未同步，所以此时HW仍未3。此后follower2同步完成后，其LEO为5，所有副本的LEO都未5，此时HW选择最小的为5。

	通过这种机制，leader副本就能知道哪些副本是满足ISR条件的（该副本LEO是否等于leader副本LEO）。