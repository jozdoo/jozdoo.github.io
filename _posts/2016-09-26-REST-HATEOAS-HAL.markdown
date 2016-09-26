---
layout: post
title:  "REST HATEOAS教程(二)：HATEOAS规范"
date:   2016-09-26
categories: REST
---


HATEOAS 是 REST(Representational state transfer) 的约束之一,它是一种思想或者是一种要求，但没有告诉我们如何去做。

### HATEOAS规范

JSON作为流行的数据交换格式，产生了许多为其定做的HATEOAS规范，下面是可以采用的规范

1. [HAL](https://tools.ietf.org/html/draft-kelly-json-hal-08)
2. [Siren](https://github.com/kevinswiber/siren)
3. [Collection](http://amundsen.com/media-types/collection/)
4. [JSON-LD](http://json-ld.org/)

其中HAL已经被Spring-HATEOAS所实现,可以很方便的在Spring-MVC的基础上进行修改，后续也会对Spring-HATEOAS进行说明。

### HAL（Hypertext Application Language)说明

HAL是为超媒体控制(HATEOAS）而建立的约定

#### HAL文档

HAL文档拥有其媒体类型 "application/hal+json",它的root必须为资源对象，例如:

~~~

   GET /orders/523 HTTP/1.1
   Host: example.org
   Accept: application/hal+json

   HTTP/1.1 200 OK
   Content-Type: application/hal+json

   {
     "_links": {
       "self": { "href": "/orders/523" },
       "warehouse": { "href": "/warehouse/56" },
       "invoice": { "href": "/invoices/873" }
     },
     "currency": "USD",
     "status": "shipped",
     "total": 10.20
   }

~~~

这个文档中的资源有 currency,status,total 三个属性,以及 _links 保留属性，其中_links 中包含链接至其他资源的信息.


#### HAL保留属性

##### _links

属性 "_links" 是可选。它是一个对象，它包含了一个或多个属性，每个属性可以是对象或者数组。

##### _embedded

属性 "_embedded"是可选的。它是一个对象，它包含了一个或多个属性，每个属性是可以对象或者数组。


#### Link属性

_links的每个属性都一个超链接对象，这些对象包含了资源至URL，他们有下列几个属性

1. href --string 必填项，它的内容可以是URL 或者URL模板。
2. templated --bool 可选项，默认为false，如果href是URL模板则templated必须为true
3. type --string 可选项，它用来表示资源类型
4. deprecation --string 可选项，表示该对象会在未来废弃
5. name --string 可选项， 可能当作第二个key，当需要选择拥有相同name的链接时
6. profile --string 可选项，简要说明链接的内容
7. title --string 可选项，用用户可读的语音来描述资源的主题
8. hreflang --string 可选项，用来表明资源的语言

下面是一个文档例子

~~~

   GET /orders HTTP/1.1
   Host: example.org
   Accept: application/hal+json

   HTTP/1.1 200 OK
   Content-Type: application/hal+json

   {
     "_links": {
       "self": { "href": "/orders" },
       "next": { "href": "/orders?page=2" },
       "find": { "href": "/orders{?id}", "templated": true }
     },
     "_embedded": {
       "orders": [{
           "_links": {
             "self": { "href": "/orders/123" },
             "basket": { "href": "/baskets/98712" },
             "customer": { "href": "/customers/7809" }
           },
           "total": 30.00,
           "currency": "USD",
           "status": "shipped",
         },{
           "_links": {
             "self": { "href": "/orders/124" },
             "basket": { "href": "/baskets/97213" },
             "customer": { "href": "/customers/12369" }
           },
           "total": 20.00,
           "currency": "USD",
           "status": "processing"
       }]
     },
     "currentlyProcessing": 14,
     "shippedToday": 20
   }

~~~





### 参考资料

[N Hypertext Application Language
                        draft-kelly-json-hal-08](https://tools.ietf.org/html/draft-kelly-json-hal-08)

[HAL - Hypertext Application Language](http://stateless.co/hal_specification.html)

[wikipedia Hypertext Application Language](https://en.wikipedia.org/wiki/Hypertext_Application_Language)

[The HAL Browser](http://haltalk.herokuapp.com/explorer/browser.html#/)
