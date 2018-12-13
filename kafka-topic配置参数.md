### 1、Create Topic
| Property(属性)                 | Default(默认值)     | Server Default Property(server.properties)说明 | 说明                                                         |
| :----------------------------- | ------------------- | ---------------------------------------------- | ------------------------------------------------------------ |
| Topic                          |                     |                                                |                                                              |
| Partitions                     | 分区                |                                                |                                                              |
| Replication Factor             | 复制因子            |                                                |                                                              |
| retention.ms                   |                     |                                                |                                                              |
| max.message.bytes              |                     |                                                |                                                              |
| segment.index.bytes            |                     |                                                |                                                              |
| segment.bytes                  |                     |                                                |                                                              |
| min.cleanable.dirty.ratio      |                     |                                                |                                                              |
| min.insync.replicas            |                     |                                                |                                                              |
| delete.retention.ms            | 86400000 (24 hours) | log.cleaner.delete.retention.ms                | 对于压缩的日志保留的最长时间，也是客户端消费消息的最长时间，同log.retention.minutes的区别在于一个控制未压缩数据，一个控制压缩后的数据。会被topic创建时的指定参数覆盖 |
| flush.messages                 | None                | log.flush.interval.messages                    | log文件”sync”到磁盘之前累积的消息条数,因为磁盘IO操作是一个慢操作,但又是一个”数据可靠性"的必要手段,所以此参数的设置,需要在"数据可靠性"与"性能"之间做必要的权衡.如果此值过大,将会导致每次"fsync"的时间较长(IO阻塞),如果此值过小,将会导致"fsync"的次数较多,这也意味着整体的client请求有一定的延迟.物理server故障,将会导致没有fsync的消息丢失. |
| preallocate                    |                     |                                                |                                                              |
| retention.bytes                |                     |                                                |                                                              |
| flush.ms                       | None                | log.flush.interval.ms                          |                                                              |
| cleanup.policy                 | delete              | log.cleanup.policy                             | 日志清理策略选择有：delete和compact主要针对过期数据的处理，或是日志文件达到限制的额度，会被  topic创建时的指定参数覆盖 |
| file.delete.delay.ms           |                     |                                                |                                                              |
| segment.jitter.ms              |                     |                                                |                                                              |
| index.interval.bytes           |                     |                                                |                                                              |
| compression.type               |                     |                                                |                                                              |
| segment.ms                     |                     |                                                |                                                              |
| unclean.leader.election.enable |                     |                                                |                                                              |

