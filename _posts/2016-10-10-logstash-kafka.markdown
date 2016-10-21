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


### 参考资料

[elastic](https://www.elastic.co/)

[apache kafka](http://kafka.apache.org/)

