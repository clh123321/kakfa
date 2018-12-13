| **名词**      | **解释**                                                     |
| ------------- | ------------------------------------------------------------ |
| Producer      | 消息的生成者                                                 |
| Consumer      | 消息的消费者                                                 |
| ConsumerGroup | 消费者组，可以并行消费Topic中的partition的消息               |
| Broker        | 缓存代理，Kafka集群中的一台或多台服务器统称broker.           |
| Topic         | Kafka处理资源的消息源(feeds of messages)的不同分类           |
| Partition     | Topic物理上的分组，一个topic可以分为多个partion,每个partion是一个有序的队列。partion中每条消息都会被分                                配一个 有序的Id(offset) |
| Message       | 消息，是通信的基本单位，每个producer可以向一个topic（主题）发布一些消息 |
| Producers     | 消息和数据生成者，向Kafka的一个topic发布消息的 过程叫做producers |
| Consumers     | 消息和数据的消费者，订阅topic并处理其发布的消费过程叫做consumers |
|               |                                                              |
|               |                                                              |

