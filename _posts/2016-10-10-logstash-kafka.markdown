---
layout: post
title:  "Logstash 集成 kafka 收集日志 笔记 (二) 搭建kafka"
date:   2016-10-21
categories: ELK logstash kafka
---

参考 [kafka quick start](http://kafka.apache.org/documentation#quickstart) 搭建kafka服务 

使用 [kafka output](https://www.elastic.co/guide/en/logstash/current/plugins-outputs-kafka.html) 将events 写入kafka topic

logstash myconfig.conf 

~~~
output {
#stdout{codec => rubydebug}
    kafka{
        topic_id => "test"
        bootstrap_servers => "localhost:9092"
    }
}

~~~

input 使用 file input  具体使用参考 [file](https://www.elastic.co/guide/en/logstash/current/plugins-inputs-file.html)

下面是input 的配置

~~~
input {
    file {
        path => "/path/to/file/*.log"
    }
}
~~~

这样logstash 就完成收集数据并写入 kafka了。

接下来是java client 接收kafka的数据。

导入

~~~
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>0.10.1.0</version>
</dependency>
~~~

参考[Class KafkaConsumer<K,V>](http://kafka.apache.org/0101/javadoc/index.html?org/apache/kafka/clients/consumer/KafkaConsumer.html) 以下为java 消费main方法代码

~~~
public static void main(String[] args) {
        Properties props = new Properties();
        props.put("bootstrap.servers", "localhost:9092");
        props.put("group.id", "test");
        props.put("enable.auto.commit", "true");
        props.put("auto.commit.interval.ms", "1000");
        props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
        consumer.subscribe(Arrays.asList("test"));
        while (true) {
            ConsumerRecords<String, String> records = consumer.poll(100);
            for (ConsumerRecord<String, String> record : records)
                System.out.printf("offset = %d, key = %s, value = %s%n", record.offset(), record.key(), record.value());
        }
    }
~~~


### 参考资料

[elastic](https://www.elastic.co/)

[apache kafka](http://kafka.apache.org/)

[kafka 0.10.1.0 API](http://kafka.apache.org/0101/javadoc/index.html)

