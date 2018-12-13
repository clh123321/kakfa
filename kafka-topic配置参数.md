### 1、Create Topic 级别配置属性表 

| Property(属性)                 | Default(默认值)     | Server Default Property(server.properties)说明 | 说明                                                         |
| :----------------------------- | ------------------- | ---------------------------------------------- | ------------------------------------------------------------ |
| Topic                          |                     |                                                |                                                              |
| Partitions                     | 分区                |                                                |                                                              |
| Replication Factor             | 复制因子            |                                                |                                                              |
| retention.ms                   | None                | log.retention.minutes                          | 数据存储的最大时间超过这个时间会根据log.cleanup.policy设置的策略处理数据，也就是消费端能够多久去消费数据 |
| max.message.bytes              | 1000012             |                                                | Kafka允许的最大的记录批次大小。如果这是增长的并且有老于0.10.2的消费者，消费者获取大小也必须是增长的以便它们可以这个大小的记录批次。在最新的消息格式版本中，记录常常分组为批次来实现更高效。在先前的消息格式版本中，未压缩的记录没有按批次分组，并且在这种情况下这样的局限仅适用于一个单一的记录。 |
| segment.index.bytes            | 10 MB               | log.index.size.max.bytes                       | 对于segment日志的索引文件大小限制，会被topic创建时的指定参数覆盖 |
| segment.bytes                  | 1 GB                | log.segment.bytes                              | topic的分区是以一堆segment文件存储的，这个控制每个segment的大小，会被topic创建时的指定参数覆盖 |
| min.cleanable.dirty.ratio      | 0.5                 | log.cleaner.min.cleanable.ratio                | 日志清理的频率控制，越大意味着更高效的清理，同时会存在一些空间上的浪费，会被topic创建时的指定参数覆盖 |
| min.insync.replicas            | 1                   |                                                | 当一个生产者设置acks 为 “all”或是 “-1”。这个配置指定了必须被认为是成功的一个写入的副本的最小数量。如果这个最小数量不能被满足，那么生产者将抛出一个异常（NotEnoughReplicas 或NotEnoughReplicasAfterAppend）。当min.insync.replicas 和 acks 一起使用时，允许你实现更强的耐久性保证。一个典型的情况是与3个复制因子创建主题，设置min.insync.replicas 为2,并且生产者的acks 为“all”。这将确保生产者抛出一个意外，如果一个主要的副本没有接收到一个写入。 |
| delete.retention.ms            | 86400000 (24 hours) | log.cleaner.delete.retention.ms                | 对于压缩的日志保留的最长时间，也是客户端消费消息的最长时间，同log.retention.minutes的区别在于一个控制未压缩数据，一个控制压缩后的数据。会被topic创建时的指定参数覆盖 |
| flush.messages                 | None                | log.flush.interval.messages                    | log文件”sync”到磁盘之前累积的消息条数,因为磁盘IO操作是一个慢操作,但又是一个”数据可靠性"的必要手段,所以此参数的设置,需要在"数据可靠性"与"性能"之间做必要的权衡.如果此值过大,将会导致每次"fsync"的时间较长(IO阻塞),如果此值过小,将会导致"fsync"的次数较多,这也意味着整体的client请求有一定的延迟.物理server故障,将会导致没有fsync的消息丢失. |
| preallocate                    | false               |                                                | 设为true时我们应当在创建一个新的日志片段时预分配文件在磁盘上。 |
| retention.bytes                | None                | log.retention.bytes                            | topic每个分区的最大文件大小，一个topic的大小限制  = 分区数*log.retention.bytes。-1没有大小限log.retention.bytes和log.retention.minutes任意一个达到要求，都会执行删除，会被topic创建时的指定参数覆盖 |
| flush.ms                       | None                | log.flush.interval.ms                          | 仅仅通过interval来控制消息的磁盘写入时机,是不足的.此参数用于控制"fsync"的时间间隔,如果消息量始终没有达到阀值,但是离上一次磁盘同步的时间间隔达到阀值,也将触发. |
| cleanup.policy                 | delete              | log.cleanup.policy                             | 日志清理策略选择有：delete和compact主要针对过期数据的处理，或是日志文件达到限制的额度，会被  topic创建时的指定参数覆盖 |
| file.delete.delay.ms           | 60000               |                                                | 从文件系统删除一个文件之前的等待时间。                       |
| segment.jitter.ms              | 0                   |                                                | 计划的片段展开时间减少的最大随机抖动，以免发生迅速集中的片段展开。 |
| index.interval.bytes           | 4096                | log.index.interval.bytes                       | 当执行一个fetch操作后，需要一定的空间来扫描最近的offset大小，设置越大，代表扫描速度越快，但是也更好内存，一般情况下不需要搭理这个参数 |
| compression.type               | producer            |                                                | 为一个给定的主题指定最终的压缩类型。这个配置接受标准的压缩编码器('gzip', 'snappy', lz4)。它额外的接受’uncompressed’，等于不压缩；还有'producer' 表示使用生产者设置的原始压缩编码器。 |
| segment.ms                     | 604800000           |                                                | 这个配置控置了时间周期，在这之后 kafka将会强制日志展开，即使这个片段文件没有满到确保保留可以删除或是压缩旧的数据。 |
| unclean.leader.election.enable | false               |                                                | 指示是否开启不在ISR里的副本设置为选举为领导者作为最后的手段，尽管这样做会导致数据丢失。 |

[参考-Kafka中Topic级别配置](https://blog.csdn.net/dly1580854879/article/details/71404135)

[kafka topic配置(超详细的核心配置项说明)](https://blog.csdn.net/myhes/article/details/83246713)

[kafka中文文档topic配置参数](https://bewithme.iteye.com/blog/2395202)

