### 生产者

##### 1，Maven依赖

```
        <dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka</artifactId>
            <version>1.3.5.RELEASE</version>
        </dependency>
```
##### 2，spring boot 配置文件

```

#============== kafka ===================
task.run.adstatisticstask=true
app.kafak.topic.name=ep.service.ad.statistics
# 指定kafka 代理地址，可以多个
spring.kafka.bootstrap-servers=172.17.11.1:9092,172.17.11.2:9092,172.17.11.3:9092,172.17.11.4:9092
#=============== provider 生产者的配置，大部分我们可以使用默认的，这里列出几个比较重要的属性 =======================
#spring.kafka.producer.retries=1
# 每次批量发送消息的数量
spring.kafka.producer.batch-size=16384
#设置大于0的值将使客户端重新发送任何数据，一旦这些数据发送失败。注意，这些重试与客户端接收到发送错误时的重试没有什么不同。允许重试将潜在的改变数据的顺序，如果这两个消息记录都是发送到同一个partition，则第一个消息失败第二个发送成功，则第二条消息会比第一条消息出现要早。
spring.kafka.producer.retries=0
#producer可以用来缓存数据的内存大小。如果数据产生速度大于向broker发送的速度，producer会阻塞或者抛出异常，以“block.on.buffer.full”来表明。这项设置将和producer能够使用的总内存相关，但并不是一个硬性的限制，因为不是producer使用的所有内存都是用于缓存。一些额外的内存会用于压缩（如果引入压缩机制），同样还有一些用于维护请求。
spring.kafka.producer.buffer-memory=33554432
# 指定消息key和消息体的编解码方式
spring.kafka.producer.key-serializer=org.apache.kafka.common.serialization.StringSerializer
spring.kafka.producer.value-serializer=org.apache.kafka.common.serialization.StringSerializer
#=============== consumer  =======================
# 指定默认消费者group id
spring.kafka.consumer.group-id=group.batch.task
#Kafka中没有初始偏移或如果当前偏移在服务器上不再存在时,默认区最新 ，有三个选项 【latest, earliest, none】
spring.kafka.consumer.auto-offset-reset=earliest
#是否开启自动提交
spring.kafka.consumer.enable-auto-commit=false
spring.kafka.consumer.ack-mode=MANUAL_IMMEDIATE
# 批量一次最大拉取数据量
spring.kafka.consumer.max-poll-records=5
# 自动提交的时间间隔
spring.kafka.consumer.auto-commit-interval=5
# 指定消息key和消息体的编解码方式
spring.kafka.consumer.key-deserializer=org.apache.kafka.common.serialization.StringDeserializer
spring.kafka.consumer.value-deserializer=org.apache.kafka.common.serialization.StringDeserializer
```
##### 3，生产者

```
@RestController
@RequestMapping("/statistics")
public class StatisticsController {

    @Value("${app.kafak.topic.name}")
    private String topicName;

    @Autowired
    private KafkaTemplate<String, String> kafkaTemplate;

    /**
     * 说明
     *
     * @param param
     * @return
     */
    @PostMapping("/add")
    public ResponseResult<String> add(@RequestBody String param) throws Exception {
        kafkaTemplate.send(topicName, param);
        return ResultGenerator.getSuccessResult("ok");
    }
}
```

##### 4，消费者

1，KafkaConfig

```

import com.google.common.collect.Maps;
import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.common.serialization.StringDeserializer;
import org.apache.kafka.common.serialization.StringSerializer;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.kafka.annotation.EnableKafka;
import org.springframework.kafka.config.ConcurrentKafkaListenerContainerFactory;
import org.springframework.kafka.config.KafkaListenerContainerFactory;
import org.springframework.kafka.core.*;
import org.springframework.kafka.listener.AbstractMessageListenerContainer;
import org.springframework.kafka.listener.ConcurrentMessageListenerContainer;

import java.util.HashMap;
import java.util.Map;

/**
 * @author: caolh
 * @description: KafkaConfig
 * @date: 2018/12/17 14:54
 */
@Configuration
@EnableKafka
public class KafkaConfig {
    @Value("${spring.kafka.bootstrap-servers}")
    private String bootstrapServers;
    @Value("${spring.kafka.consumer.enable-auto-commit}")
    private Boolean autoCommit;
    @Value("${spring.kafka.consumer.auto-commit-interval}")
    private Integer autoCommitInterval;
    @Value("${spring.kafka.consumer.group-id}")
    private String groupId;
    @Value("${spring.kafka.consumer.max-poll-records}")
    private Integer maxPollRecords;
    @Value("${spring.kafka.consumer.auto-offset-reset}")
    private String autoOffsetReset;
    @Value("${spring.kafka.producer.retries}")
    private Integer retries;
    @Value("${spring.kafka.producer.batch-size}")
    private Integer batchSize;
    @Value("${spring.kafka.producer.buffer-memory}")
    private Integer bufferMemory;

    /**
     * 生产者配置信息
     */
    @Bean
    public Map<String, Object> producerConfigs() {
        Map<String, Object> props = Maps.newHashMap();
        props.put(ProducerConfig.ACKS_CONFIG, "0");
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
        props.put(ProducerConfig.RETRIES_CONFIG, retries);
        props.put(ProducerConfig.BATCH_SIZE_CONFIG, batchSize);
        props.put(ProducerConfig.LINGER_MS_CONFIG, 1);
        props.put(ProducerConfig.BUFFER_MEMORY_CONFIG, bufferMemory);
        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        return props;
    }

    /**
     * 生产者工厂
     */
    @Bean
    public ProducerFactory<String, String> producerFactory() {
        return new DefaultKafkaProducerFactory<>(producerConfigs());
    }

    /**
     * 生产者模板
     */
    @Bean
    public KafkaTemplate<String, String> kafkaTemplate() {
        return new KafkaTemplate<>(producerFactory());
    }

    /**
     * 消费者配置信息
     */
    @Bean
    public Map<String, Object> consumerConfigs() {
        Map<String, Object> props = Maps.newHashMap();
        props.put(ConsumerConfig.GROUP_ID_CONFIG, groupId);
        props.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, autoOffsetReset);
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
        props.put(ConsumerConfig.MAX_POLL_RECORDS_CONFIG, maxPollRecords);
        props.put(ConsumerConfig.SESSION_TIMEOUT_MS_CONFIG, 120000);
        props.put(ConsumerConfig.REQUEST_TIMEOUT_MS_CONFIG, 180000);
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        return props;
    }

    /**
     * 消费者批量工程
     */
    @Bean
    public KafkaListenerContainerFactory<?> batchFactory() {
        ConcurrentKafkaListenerContainerFactory<Integer, String> factory = new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(new DefaultKafkaConsumerFactory<>(consumerConfigs()));
        //factory.setConcurrency(1);
        //设置为批量消费，每个批次数量在Kafka配置参数中设置ConsumerConfig.MAX_POLL_RECORDS_CONFIG
        factory.setBatchListener(true);
        //factory.getContainerProperties().setAckMode(AbstractMessageListenerContainer.AckMode.MANUAL_IMMEDIATE);//设置提交偏移量的方式
        return factory;
    }
}
```

2，消费

```
/**
 * @author: caolh
 * @description: AdStatisticsTask
 * @date: 2018/12/12 15:01
 */
@Component
@ConditionalOnProperty(name = "task.run.adstatisticstask", havingValue = "true")
public class AdStatisticsTask extends BaseTask {
    //{@TopicPartition(topic = "topic1", partitions = { "0", "1" }))}
    //@KafkaListener(group = "group.logstash", topics = {"${app.kafak.topic.name}"})
    //@KafkaListener(group = "group_1", topicPartitions = {@TopicPartition(topic = "${app.kafak.topic.name}", partitions = {"0"})})
    /**
     * @author caolh
     * @description: 批量
     * @date:2018/12/17 16:01
     * @return
     */
    @KafkaListener(topics = "${app.kafak.topic.name}", containerFactory = "batchFactory")
    protected void process(List<ConsumerRecord<String, String>> message) throws InterruptedException {
        System.out.println(new Date());
        System.out.println("--------------------.size" + message.size());
        for (ConsumerRecord<String, String> consumerRecord : message) {
            System.out.println(consumerRecord.offset());
        }
        Thread.sleep(1000L);
    }
    /**
     * @author caolh
     * @description: 单条
     * @date:2018/12/17 16:05
     * @return 
     */
    @KafkaListener(topics = "${app.kafak.topic.name}")
    protected void process(ConsumerRecord<String, String> record) throws InterruptedException {
        System.out.println("++++");
        System.out.println(record.offset());
        Thread.sleep(1000L);
    }


//    /**
//     * /**
//     * 监听topic1主题,单条消费
//     */
//    @KafkaListener(topics = "topic1")
//    public void listen0(ConsumerRecord<String, String> record) {
//        consumer(record);
//    }
//
//    /**
//     * 监听topic2主题,单条消费
//     */
//    @KafkaListener(topics = "${topicName.topic2}")
//    public void listen1(ConsumerRecord<String, String> record) {
//        consumer(record);
//    }
//
//    /**
//     * 监听topic3和topic4,单条消费
//     */
//    @KafkaListener(topics = {"topic3", "topic4"})
//    public void listen2(ConsumerRecord<String, String> record) {
//        consumer(record);
//    }
//
//    /**
//     * 监听topic5,批量消费
//     */
//    @KafkaListener(topics = "${topicName.topic2}", containerFactory = "batchFactory")
//    public void listen2(List<ConsumerRecord<String, String>> records) {
//        batchConsumer(records);
//    }
//
//    /**
//     * 单条消费
//     */
//    public void consumer(ConsumerRecord<String, String> record) {
//        //logger.debug("主题:{}, 内容: {}", record.topic(), record.value());
//    }
//
//    /**
//     * 批量消费
//     */
//    public void batchConsumer(List<ConsumerRecord<String, String>> records) {
//        //records.forEach(record -> consumer(record));
//    }

}
```

##### 5，KafkaConsumer 消费者使用

1，KafkaConsumer 

```
/**
 * @author: caolh
 * @description: KafkaConnectConfig
 * @date: 2018/12/17 16:18
 */
@Configuration
@ConfigurationProperties(prefix = "spring.kafka")
public class KafkaConnectConfig {

    @Bean(name = "kafkaConsumer")
    public KafkaConsumer<String, String> kafkaConsumer() {
        Properties props = new Properties();
        props.put("bootstrap.servers", bootstrapServers);
        props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        props.setProperty("group.id", "ggg");
        props.setProperty("enable.auto.commit", enableAutoCommit);
        props.setProperty("auto.offset.reset", autoOffsetReset);
        KafkaConsumer<String, String> consumer = new KafkaConsumer<String, String>(props);
        consumer.subscribe(Arrays.asList("gzj"));

        return consumer;
    }
    @Value("${server.port}")
    private String port;
    @Value("${spring.kafka.consumer.group-id}")
    private String groupId;
    @Value("${spring.kafka.bootstrap-servers}")
    private String bootstrapServers;
    @Value("${spring.kafka.consumer.enable-auto-commit}")
    private String enableAutoCommit;
    @Value("${spring.kafka.consumer.auto-offset-reset}")
    private String autoOffsetReset;

    public String getGroupId() {
        return groupId;
    }

    public void setGroupId(String groupId) {
        this.groupId = groupId;
    }

    public String getBootstrapServers() {
        return bootstrapServers;
    }

    public void setBootstrapServers(String bootstrapServers) {
        this.bootstrapServers = bootstrapServers;
    }


    public String getEnableAutoCommit() {
        return enableAutoCommit;
    }

    public void setEnableAutoCommit(String enableAutoCommit) {
        this.enableAutoCommit = enableAutoCommit;
    }

    public String getAutoOffsetReset() {
        return autoOffsetReset;
    }

    public void setAutoOffsetReset(String autoOffsetReset) {
        this.autoOffsetReset = autoOffsetReset;
    }
}
```

2，消费

```
/**
 * @author: caolh
 * @description: AdStatisticsConsumerTask
 * @date: 2018/12/17 16:12
 */
@Component
@ConditionalOnProperty(name = "task.run.adstatisticconsumerstask", havingValue = "true")
public class AdStatisticsConsumerTask implements Runnable, InitializingBean {
    private static final Logger logger = LoggerFactory.getLogger(AdStatisticsConsumerTask.class);
    private Thread thread;

    private static final String q = "kafkaConsumer";

    @Resource(name = q)
    private KafkaConsumer<String, String> kafkaConsumer;

    @Override
    public void run() {
        logger.info("消费数据任务启动");
        while (true) {
            try {
                ConsumerRecords<String, String> records = kafkaConsumer.poll(1000);
                if (records != null) {
                    for (ConsumerRecord<String, String> record : records) {
                        logger.error(record.key());
                        logger.error(record.topic());
                        logger.error(record.value());
                    }
                }
            } catch (Exception e) {
                logger.error("我也不知道哪儿错了");
            } finally {
                logger.error("不放弃");
            }
        }
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        this.thread = new Thread(this);
        this.thread.start();
    }
}
```

