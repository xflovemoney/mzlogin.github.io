---
layout: post
title:  Kafka Java API示例
categories: 分布式
description: 分布式
keywords: 分布式
---
jar 依赖包
http://download.csdn.net/download/alexander_zhou/9192011

1、Producer端

```java
    import java.util.Properties;  
      
    import kafka.javaapi.producer.Producer;  
    import kafka.producer.KeyedMessage;  
    import kafka.producer.ProducerConfig;  
      
    public class KafkaProducer{  
          
        private final Producer<String, String> producer;  
        public final static String TOPIC = "testtopic";  
      
        private KafkaProducer(){  
              
            Properties props = new Properties();  
              
            // 此处配置的是kafka的broker地址:端口列表  
            props.put("metadata.broker.list", "192.168.1.225:9092,192.168.1.226:9092");  
      
            //配置value的序列化类  
            props.put("serializer.class", "kafka.serializer.StringEncoder");  
              
            //配置key的序列化类  
            props.put("key.serializer.class", "kafka.serializer.StringEncoder");  
      
            //request.required.acks  
            //0, which means that the producer never waits for an acknowledgement from the broker (the same behavior as 0.7). This option provides the lowest latency but the weakest durability guarantees (some data will be lost when a server fails).  
            //1, which means that the producer gets an acknowledgement after the leader replica has received the data. This option provides better durability as the client waits until the server acknowledges the request as successful (only messages that were written to the now-dead leader but not yet replicated will be lost).  
            //-1, which means that the producer gets an acknowledgement after all in-sync replicas have received the data. This option provides the best durability, we guarantee that no messages will be lost as long as at least one in sync replica remains.  
            props.put("request.required.acks","-1");  
      
            producer = new Producer<String, String>(new ProducerConfig(props));  
        }  
      
        void produce() {  
            int messageNo = 1;  
            final int COUNT = 101;  
      
            int messageCount = 0;  
            while (messageNo < COUNT) {  
                String key = String.valueOf(messageNo);  
                String data = "Hello kafka message :" + key;  
                producer.send(new KeyedMessage<String, String>(TOPIC, key ,data));  
                System.out.println(data);  
                messageNo ++;  
                messageCount++;  
            }  
              
            System.out.println("Producer端一共产生了" + messageCount + "条消息！");  
        }  
      
        public static void main( String[] args )  
        {  
            new KafkaProducer().produce();  
        }  
    }  
```
运行结果：
```java
Hello kafka message :1  
Hello kafka message :2  
Hello kafka message :3  
Hello kafka message :4  
Hello kafka message :5  
Hello kafka message :6  
Hello kafka message :7  
Hello kafka message :8  
Hello kafka message :9  
Hello kafka message :10  
Hello kafka message :11  
Hello kafka message :12  
Hello kafka message :13  
Hello kafka message :14  
Hello kafka message :15  
Hello kafka message :16  
Hello kafka message :17  
Hello kafka message :18  
Hello kafka message :19  
Hello kafka message :20  
Hello kafka message :21  
Hello kafka message :22  
Hello kafka message :23  
Hello kafka message :24  
Hello kafka message :25  
Hello kafka message :26  
Hello kafka message :27  
Hello kafka message :28  
Hello kafka message :29  
Hello kafka message :30  
Hello kafka message :31  
Hello kafka message :32  
Hello kafka message :33  
Hello kafka message :34  
Hello kafka message :35  
Hello kafka message :36  
Hello kafka message :37  
Hello kafka message :38  
Hello kafka message :39  
Hello kafka message :40  
Hello kafka message :41  
Hello kafka message :42  
Hello kafka message :43  
Hello kafka message :44  
Hello kafka message :45  
Hello kafka message :46  
Hello kafka message :47  
Hello kafka message :48  
Hello kafka message :49  
Hello kafka message :50  
Hello kafka message :51  
Hello kafka message :52  
Hello kafka message :53  
Hello kafka message :54  
Hello kafka message :55  
Hello kafka message :56  
Hello kafka message :57  
Hello kafka message :58  
Hello kafka message :59  
Hello kafka message :60  
Hello kafka message :61  
Hello kafka message :62  
Hello kafka message :63  
Hello kafka message :64  
Hello kafka message :65  
Hello kafka message :66  
Hello kafka message :67  
Hello kafka message :68  
Hello kafka message :69  
Hello kafka message :70  
Hello kafka message :71  
Hello kafka message :72  
Hello kafka message :73  
Hello kafka message :74  
Hello kafka message :75  
Hello kafka message :76  
Hello kafka message :77  
Hello kafka message :78  
Hello kafka message :79  
Hello kafka message :80  
Hello kafka message :81  
Hello kafka message :82  
Hello kafka message :83  
Hello kafka message :84  
Hello kafka message :85  
Hello kafka message :86  
Hello kafka message :87  
Hello kafka message :88  
Hello kafka message :89  
Hello kafka message :90  
Hello kafka message :91  
Hello kafka message :92  
Hello kafka message :93  
Hello kafka message :94  
Hello kafka message :95  
Hello kafka message :96  
Hello kafka message :97  
Hello kafka message :98  
Hello kafka message :99  
Hello kafka message :100  
Producer端一共产生了100条消息！ 
```
2、Consumer端
```java
    import java.util.HashMap;  
    import java.util.List;  
    import java.util.Map;  
    import java.util.Properties;  
      
    import kafka.consumer.ConsumerConfig;  
    import kafka.consumer.ConsumerIterator;  
    import kafka.consumer.KafkaStream;  
    import kafka.javaapi.consumer.ConsumerConnector;  
    import kafka.serializer.StringDecoder;  
    import kafka.utils.VerifiableProperties;  
      
    public class KafkaConsumer {  
      
        private final ConsumerConnector consumer;  
      
        private KafkaConsumer() {  
            Properties props = new Properties();  
              
            // zookeeper 配置  
            props.put("zookeeper.connect", "server3:2181");  
      
            // 消费者所在组  
            props.put("group.id", "testgroup");  
      
            // zk连接超时  
            props.put("zookeeper.session.timeout.ms", "4000");  
            props.put("zookeeper.sync.time.ms", "200");  
            props.put("auto.commit.interval.ms", "1000");  
            props.put("auto.offset.reset", "smallest");  
              
            // 序列化类  
            props.put("serializer.class", "kafka.serializer.StringEncoder");  
      
            ConsumerConfig config = new ConsumerConfig(props);  
      
            consumer = kafka.consumer.Consumer.createJavaConsumerConnector(config);  
        }  
      
        void consume() {  
            Map<String, Integer> topicCountMap = new HashMap<String, Integer>();  
            topicCountMap.put(KafkaProducer.TOPIC, new Integer(1));  
      
            StringDecoder keyDecoder = new StringDecoder(new VerifiableProperties());  
            StringDecoder valueDecoder = new StringDecoder(new VerifiableProperties());  
      
            Map<String, List<KafkaStream<String, String>>> consumerMap =   
                    consumer.createMessageStreams(topicCountMap,keyDecoder,valueDecoder);  
            KafkaStream<String, String> stream = consumerMap.get(KafkaProducer.TOPIC).get(0);  
            ConsumerIterator<String, String> it = stream.iterator();  
              
            int messageCount = 0;  
            while (it.hasNext()){  
                System.out.println(it.next().message());  
                messageCount++;  
                if(messageCount == 100){  
                    System.out.println("Consumer端一共消费了" + messageCount + "条消息！");  
                }  
            }  
        }  
      
        public static void main(String[] args) {  
            new KafkaConsumer().consume();  
        }  
    }  
```
运行结果：
```java
    Hello kafka message :3  
    Hello kafka message :8  
    Hello kafka message :14  
    Hello kafka message :19  
    Hello kafka message :23  
    Hello kafka message :28  
    Hello kafka message :32  
    Hello kafka message :37  
    Hello kafka message :41  
    Hello kafka message :46  
    Hello kafka message :50  
    Hello kafka message :55  
    Hello kafka message :64  
    Hello kafka message :69  
    Hello kafka message :73  
    Hello kafka message :78  
    Hello kafka message :82  
    Hello kafka message :87  
    Hello kafka message :91  
    Hello kafka message :96  
    Hello kafka message :2  
    Hello kafka message :7  
    Hello kafka message :13  
    Hello kafka message :18  
    Hello kafka message :22  
    Hello kafka message :27  
    Hello kafka message :31  
    Hello kafka message :36  
    Hello kafka message :40  
    Hello kafka message :45  
    Hello kafka message :54  
    Hello kafka message :59  
    Hello kafka message :63  
    Hello kafka message :68  
    Hello kafka message :72  
    Hello kafka message :77  
    Hello kafka message :81  
    Hello kafka message :86  
    Hello kafka message :90  
    Hello kafka message :95  
    Hello kafka message :100  
    Hello kafka message :5  
    Hello kafka message :11  
    Hello kafka message :16  
    Hello kafka message :20  
    Hello kafka message :25  
    Hello kafka message :34  
    Hello kafka message :39  
    Hello kafka message :43  
    Hello kafka message :48  
    Hello kafka message :52  
    Hello kafka message :57  
    Hello kafka message :61  
    Hello kafka message :66  
    Hello kafka message :70  
    Hello kafka message :75  
    Hello kafka message :84  
    Hello kafka message :89  
    Hello kafka message :93  
    Hello kafka message :98  
    Hello kafka message :4  
    Hello kafka message :9  
    Hello kafka message :10  
    Hello kafka message :15  
    Hello kafka message :24  
    Hello kafka message :29  
    Hello kafka message :33  
    Hello kafka message :38  
    Hello kafka message :42  
    Hello kafka message :47  
    Hello kafka message :51  
    Hello kafka message :56  
    Hello kafka message :60  
    Hello kafka message :65  
    Hello kafka message :74  
    Hello kafka message :79  
    Hello kafka message :83  
    Hello kafka message :88  
    Hello kafka message :92  
    Hello kafka message :97  
    Hello kafka message :1  
    Hello kafka message :6  
    Hello kafka message :12  
    Hello kafka message :17  
    Hello kafka message :21  
    Hello kafka message :26  
    Hello kafka message :30  
    Hello kafka message :35  
    Hello kafka message :44  
    Hello kafka message :49  
    Hello kafka message :53  
    Hello kafka message :58  
    Hello kafka message :62  
    Hello kafka message :67  
    Hello kafka message :71  
    Hello kafka message :76  
    Hello kafka message :80  
    Hello kafka message :85  
    Hello kafka message :94  
    Hello kafka message :99  
    Consumer端一共消费了100条消息！  
```
[原文链接](http://blog.csdn.net/lipeng_bigdata/article/details/51036099)
