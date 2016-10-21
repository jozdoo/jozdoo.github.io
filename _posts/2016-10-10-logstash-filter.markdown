---
layout: post
title:  "Logstash 集成 kafka 收集日志 笔记 (一) 过滤数据"
date:   2016-10-21
categories: ELK logstash
---

公司有一个存取调用外部计费接口细节的需求，目前已经有了这样这样一种实现，生产端调用外部计费接口，并将调用日志写入文件，利用NFS在收集服务器上挂载日志文件，通过文件操作读取文件并分析，最后写入数据库。

这样做有几点弊端：

1. 增加了网络的复杂性
2. 耦合过于紧密 ，点对点直连，很难进行自动管理
3. 不利于后期分布式扩展，如果有多个日志生成端，消费端难以处理

因此希望能够增加一层消息队列来解耦，这里采用kafka。
日志的采集采用 logstash , 因为公司已经使用了 ELK 来管理日志。


## 安装logstash
---

按照elastic 官方文档说明进行安装 

Logstash requires Java 7 or later

Download and unzip the latest logstash release

测试能否运行

~~~
bin/logstash -e 'input { stdin { } } output { stdout {} }'
~~~

~~~
hello world
2013-11-21T01:22:14.405+0000 0.0.0.0 hello world
~~~

## **编写配置文件**

分析日志格式:

~~~
[2016-10-12 22:33:29] API faceAuth 5d3f2412131231ac18ac89b064d420 {"devId":"5d3f2412131231ac18ac89b064d420","result":"4","realName":"沈匿名","idCard":"224302199****00643","supplier":"XS","time":"2016-10-12 22:33:29:309","interfaceName":"faceAuth","type":"API","message":"身份证号不存在"}
~~~

### **使用grok filter**

我的目的是将 这段日志转为JSON格式 ， 在阅读 [Getting Started with Logstash](https://www.elastic.co/guide/en/logstash/current/getting-started-with-logstash.html) 后我知道了过滤插件 grok 它可以将原始文件分割为KV对象。

这里我先使用[grok debug](http://grokdebug.herokuapp.com/) 对日志进行调试

grok 使用正则匹配，映射为KV 下面是示例


~~~
input:
(?<allContent>.*)

output:
{
  "allContent": [
    "[2016-10-12 22:33:29] API faceAuth 5d3f2412131231ac18ac89b064d420 {"devId":"5d3f2412131231ac18ac89b064d420","result":"4","realName":"沈匿名","idCard":"224302199****00643","supplier":"XS","time":"2016-10-12 22:33:29:309","interfaceName":"faceAuth","type":"API","message":"身份证号不存在"}"
  ]
}
~~~

按照以下格式依次进行匹配

~~~
(?<field_name>the pattern here)
~~~

每次进行编写大量正则还是蛮辛苦的，这里 grok 提供了很多基本正则模板上面的匹配可以改成这样 这里使用 GREEDYDATA .* 模板



~~~
input：
%{GREEDYDATA:allContent}

output：
{
  "allContent": [
    "[2016-10-12 22:33:29] API faceAuth 5d3f2412131231ac18ac89b064d420 {"devId":"5d3f2412131231ac18ac89b064d420","result":"4","realName":"沈匿名","idCard":"224302199****00643","supplier":"XS","time":"2016-10-12 22:33:29:309","interfaceName":"faceAuth","type":"API","message":"身份证号不存在"}"
  ]
}
~~~

达到一样的效果，grok提供的模板在[logstash-plugins/logstash-patterns-core](https://github.com/logstash-plugins/logstash-patterns-core/tree/master/patterns)

当然也可以有自定义的模板
自定义模板的文件内容格式如下

~~~
# contents of ./patterns/postfix:
POSTFIX_QUEUEID [0-9A-F]{10,11}
allContent .*
~~~

定义模板需要在 grok块内引入

~~~

filter {
  grok {
    patterns_dir => ["./patterns"]
    match => { "message" => "%{allContent:allContent}}
  }
}

~~~

这样我打算使用内置的模板实现我的目的，边调试变查询模板列表，我写下如下模板



~~~
input：
%{NOTSPACE}%{DATESTAMP:date}%{NOTSPACE} %{WORD} %{WORD} %{WORD} %{GREEDYDATA:data}

output：
{
  "date": [
    "6-10-12 22:33:29"
  ],
  "type": [
    "faceAuth"
  ],
  "id": [
    "5d3f2412131231ac18ac89b064d420"
  ],
  "data": [
    "{"devId":"5d3f2412131231ac18ac89b064d420","result":"4","realName":"沈匿名","idCard":"224302199****00643","supplier":"XS","time":"2016-10-12 22:33:29:309","interfaceName":"faceAuth","type":"API","message":"身份证号不存在"}"
  ]
}

~~~

### **使用json filter**

data 数据块内的JSON已经是格式化后的，内容也很多，虽然已经grok能够达到我的目的，但是这么多还是很麻烦，logstash 提供非常多的filter 我希望能找到处理JSON的filter，查阅[Filter plugins](https://www.elastic.co/guide/en/logstash/current/filter-plugins.html) 我找到了 [json](https://www.elastic.co/guide/en/logstash/current/plugins-filters-json.html}) 

json filter 使用 json{}块
主要使用其中的俩个命令  target,source

* target 定义输错至那个field 
* source 定义从那个field获取字符串，并转为json格式

tips: 如果没有定义target ，source会将json 输出至 root下

这下工具都找全了。

### **编写 filter 块**
我最后编写的 filter 块如下

~~~

filter {
    grok {
        patterns_dir => ["./patterns"]
        match => {"message" => "%{NOTSPACE}%{DATESTAMP:date}%{NOTSPACE} %{WORD} %{WORD} %{WORD} %{GREEDYDATA:data}"}
    }

    json {
        source => "data"
        remove_field => "data"
    }
    
}

~~~

### **调试 logstash**

为了方便调试 我将 intput{} 以及 output{} 都设置为命令行输入输出

~~~

input {
    stdin{}
}

filter {
    grok {
        patterns_dir => ["./patterns"]
        match => {"message" => "%{NOTSPACE}%{DATESTAMP:date}%{NOTSPACE} %{WORD} %{WORD} %{WORD} %{GREEDYDATA:data}"}
    }

    json {
        source => "data"
        remove_field => "data"
    }
}
output {
    stdout{codec => rubydebug}
}

~~~

启动logstash
 
~~~
bin/logstash -f conf/myconfig.conf
~~~

~~~
output:
{
          "message" => "身份证号不存在",
         "@version" => "1",
       "@timestamp" => "2016-10-21T03:58:02.208Z",
             "host" => "jozdoo.local",
             "date" => "6-10-12 22:33:29",
            "devId" => "5d3f2412131231ac18ac89b064d420",
           "result" => "4",
         "realName" => "沈匿名",
           "idCard" => "224302199****00643",
         "supplier" => "XS",
             "time" => "2016-10-12 22:33:29:309",
    "interfaceName" => "faceAuth",
             "type" => "API"
}
~~~

发现 input中的 message 将 原始message 覆盖了 ， 要将 input中的message 重命名为 rechargeMessage 

### **使用 mutate 重命名 message 字段**

mutate filter 可以用来 重命名，删除，替换，修改 field。

修改后的 myconfig.conf

~~~
input {
    stdin{}
}

filter {
    grok {
        patterns_dir => ["./patterns"]
        match => {"message" => "%{NOTSPACE}%{DATESTAMP:date}%{NOTSPACE} %{WORD} %{WORD} %{WORD} %{GREEDYDATA:data}"}
    }
    mutate {
        rename => { "message" => "tempMessage" }
    }
    json {
        source => "data"
        remove_field => "data"
    }
    mutate {
        rename => { "message" => "rechargeMessage" }
        rename => { "tempMessage" => "message" }
    }
}
output {
    stdout{codec => rubydebug}
}
~~~



~~~
output:
{
           "@version" => "1",
         "@timestamp" => "2016-10-21T04:36:36.439Z",
               "host" => "jozdoo.local",
               "date" => "6-10-12 22:33:29",
              "devId" => "5d3f2412131231ac18ac89b064d420",
             "result" => "4",
           "realName" => "沈匿名",
             "idCard" => "224302199****00643",
           "supplier" => "XS",
               "time" => "2016-10-12 22:33:29:309",
      "interfaceName" => "faceAuth",
               "type" => "API",
    "rechargeMessage" => "身份证号不存在",
            "message" => "[2016-10-12 22:33:29] API faceAuth 5d3f2412131231ac18ac89b064d420 {\"devId\":\"5d3f2412131231ac18ac89b064d420\",\"result\":\"4\",\"realName\":\"沈匿名\",\"idCard\":\"224302199****00643\",\"supplier\":\"XS\",\"time\":\"2016-10-12 22:33:29:309\",\"interfaceName\":\"faceAuth\",\"type\":\"API\",\"message\":\"身份证号不存在\"}"
}
~~~

接下来需要配置 input， output，kafka，以及java 消费kafka的代码





### 参考资料

[elastic](https://www.elastic.co/)

[logstash-patterns-core](https://github.com/logstash-plugins/logstash-patterns-core)

[Grok Debugger](http://grokdebug.herokuapp.com/)
