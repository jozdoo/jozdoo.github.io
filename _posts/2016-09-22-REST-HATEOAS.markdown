---
layout: post
title:  "REST HATEOAS教程(一)：HATEOAS入门"
date:   2016-09-22
categories: REST
---
HATEOAS 是 REST(Representational state transfer) 的约束之一。

*HATEOAS* 是 *Hypermedia As The Engine Of Application State* 的缩写，从字面上理解是 *“超媒体即是应用状态引擎”* 。其原则就是客户端与服务器的交互完全由超媒体动态提供，客户端无需事先了解如何与数据或者服务器交互。相反的，在一些RPC服务或者Redis,Mysql等软件，需要事先了解接口定义或者特定的交互语法。


### HATEOAS在REST中的地位

在[Richardson Maturity Model](http://martinfowler.com/articles/richardsonMaturityModel.html#level0)模型中，将RESTful分为四步,其中第四步 *Hypermedia Controls* 也就是HATEOAS。

1. 第一个层次（Level 0）的 Web 服务只是使用 HTTP 作为传输方式，实际上只是远程方法调用（RPC）的一种具体形式。SOAP 和 XML-RPC 都属于此类。
2. 第二个层次（Level 1）的 Web 服务引入了资源的概念。每个资源有对应的标识符和表达。
3. 第三个层次（Level 2）的 Web 服务使用不同的 HTTP 方法来进行不同的操作，并且使用 HTTP 状态码来表示不同的结果。如 HTTP GET 方法来获取资源，HTTP DELETE 方法来删除资源。
4. 第四个层次（Level 3）的 Web 服务使用 HATEOAS。在资源的表达中包含了链接信息。客户端可以根据链接来发现可以执行的动作。

REST的设计者Roy T. Fielding在博客 [REST APIs must be hypertext-driven](http://roy.gbiv.com/untangled/2008/rest-apis-must-be-hypertext-driven) 中的几点原则强调非HATEOAS的系统不能称为RESTful。

1. REST API决不能定义固定的资源名称或者层次关系
2. 使用REST API应该只需要知道初始URI（书签）和一系列针对目标用户的标准媒体类型

### HATEOAS 例子

通过实现HATEOAS，每个资源能够描述针对自己的操作资源，动态的控制客户端，即便更改了URL也不会破坏客户端。

下面是个例子,首先是一个GET请求

~~~
    GET /account/12345 HTTP/1.1
    Host: somebank.org
    Accept: application/xml
    ...
~~~

将会返回

~~~
   HTTP/1.1 200 OK
   Content-Type: application/xml
   Content-Length: ...

   <?xml version="1.0"?>
   <account>
      <account_number>12345</account_number>
      <balance currency="usd">100.00</balance>
      <link rel="deposit" href="https://somebank.org/account/12345/deposit" />
      <link rel="withdraw" href="https://somebank.org/account/12345/withdraw" />
      <link rel="transfer" href="https://somebank.org/account/12345/transfer" />
      <link rel="close" href="https://somebank.org/account/12345/close" />
    </account>
~~~

返回的body不仅包含了 账号信息 账户编号:12345，账号余额100同时还有四个可执行链接分别可以执行deposit,withdraw,transfer,close。

一段时间后再次查询用户信息时返回

~~~
   HTTP/1.1 200 OK
   Content-Type: application/xml
   Content-Length: ...

   <?xml version="1.0"?>
   <account>
       <account_number>12345</account_number>
       <balance currency="usd">-25.00</balance>
       <link rel="deposit" href="https://somebank.org/account/12345/deposit" />
   </account>
~~~

这时用户账号余额产生赤字，可操作链接只剩一个 deposit, 其余三个在赤字情况下无法执行。

### HATEOAS带来的好处

让API变的可读性更高 ，实现客户端与服务端的部分解耦。对于不使用 HATEOAS 的 REST 服务，客户端和服务器的实现之间是紧密耦合的。客户端需要根据服务器提供的相关文档来了解所暴露的资源和对应的操作。当服务器发生了变化时，如修改了资源的 URI，客户端也需要进行相应的修改。而使用 HATEOAS 的 REST 服务中，客户端可以通过服务器提供的资源的表达来智能地发现可以执行的操作。当服务器发生了变化时，客户端并不需要做出修改，因为资源的 URI 和其他信息都是动态发现的。

### 参考资料

[REST API Turorial](http://www.restapitutorial.com/)

[使用 Spring HATEOAS 开发 REST 服务](http://www.ibm.com/developerworks/cn/java/j-lo-SpringHATEOAS/)

[wikipedia Representational state transfer](https://en.wikipedia.org/w/index.php?title=Representational_state_transfer&gettingStartedReturn=true)

[wikipedia HATEOAS](https://en.wikipedia.org/wiki/HATEOAS)

[REST APIs must be hypertext-driven](http://roy.gbiv.com/untangled/2008/rest-apis-must-be-hypertext-driven)

[REST CookBook](http://restcookbook.com/)

[Hypertext Application Language](https://en.wikipedia.org/wiki/Hypertext_Application_Language)

[Richardson Maturity Model](http://martinfowler.com/articles/richardsonMaturityModel.html)
