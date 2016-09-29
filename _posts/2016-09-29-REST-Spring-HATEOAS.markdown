---
layout: post
title:  "REST HATEOAS教程(三)：Spring-HATEOAS实现"
date:   2016-09-29
categories: REST
---


Spring HATEOAS 提供了些API，能让我们更容易的创建出符合HATEOAS原则的Respons。

Spring HATEOAS 实现了部分HAL规范，对response的_links有一部分的支持，但任不完整。对 _embedded ,Spring HATEOAS 并没有在文档中对其进行说明。

查看其源码后，其实已经有部分关于 _embedded 的API。在**org.springframework.hateoas.core**,**org.springframework.hateoas.hal**包中有部分 _embedded的装配类， 但都还是很初步的设想并没有与spring集成。

因此如果要完全的准守 HAL 规范，只能自己实现_embedde的部分了，或者把spring HATEOAS 看着HAL的变种实现吧。

### Links

Spring HATEOAS 提供构建 link对象的API,这里实现的对象仅有俩个属性

* rel 对应HAL中link对象名
* href 即HAL中的link的href属性

#### 构造方法构建 Link

构建Link对象准备放入Resource

~~~

Link link = new Link("http://localhost:8080/something");
assertThat(link.getHref(), is("http://localhost:8080/something"));
assertThat(link.getRel(), is(Link.SELF));

Link link = new Link("http://localhost:8080/something", "my-rel");
assertThat(link.getHref(), is("http://localhost:8080/something"));
assertThat(link.getRel(), is("my-rel"));

~~~

###Resource

要实现Resource 需要POJO继承ResourceSupport类, ResourceSupport实现Identifiable<Link>接口,实现REST中**每个资源都有自己的唯一id**。 

ResourceSupport中的List<Link> links属性，用于存储构建出来的Link，并由org.springframework.hateoas.hal.ResourceSupportMixin 将ResourceSupport序列号为 _links 来匹配HAL规范，


~~~

class PersonResource extends ResourceSupport {
  String firstname;
  String lastname;
}

PersonResource resource = new PersonResource();
resource.firstname = "Dave";
resource.lastname = "Matthews";
resource.add(new Link("http://myhost/people"));

~~~

###Link Builder

####利用controller类构建

Spring HATEOAS 还提供了方便的Link构建类，可以通过Controller类直接构建

下面是普遍的Controller

~~~

@Controller
@RequestMapping("/people")
class PersonController {

  @RequestMapping(method = RequestMethod.GET)
  public HttpEntity<PersonResource> showAll() { … }

  @RequestMapping(value = "/{person}", method = RequestMethod.GET)
  public HttpEntity<PersonResource> show(@PathVariable Long person) { … }
}

~~~

通过Controller 构建

~~~

import static org.sfw.hateoas.mvc.ControllerLinkBuilder.*;

Link link = linkTo(PersonController.class).withRel("people");
assertThat(link.getRel(), is("people"));
assertThat(link.getHref(), endsWith("/people"));

~~~

为 Link 的 href 附上Resource 的id

~~~

Person person = new Person(1L, "Dave", "Matthews");
//                 /person                 /     1
Link link = linkTo(PersonController.class).slash(person.getId()).withSelfRel();
assertThat(link.getRel(), is(Link.SELF));
assertThat(link.getHref(), endsWith("/people/1"));

~~~

public T slash(Object object) 会调用实现Identifiable接口的object的getId()，添加至href尾部

~~~

class Person implements Identifiable<Long> {
  public Long getId() { … }
}

Link link = linkTo(PersonController.class).slash(person).withSelfRel();

~~~

####利用Controller Method 构建

~~~

Link link = linkTo(methodOn(PersonController.class).show(2L)).withSelfRel();
assertThat(link.getHref(), is("/people/2")));

~~~ 

创建的href会根据 @PathVariable 动态生成

####EntityLinks

EntityLinks 接口的实现能够返回 集合资源 (/people) 或者 单个资源 (/pelple/1)，使用@EnableEntityLinks 注入EntityLinks实例，@ExposesResourceFor(…)注解绑定Controller及资源类，但是在使用EntityLinks 需要注意俩点

1. Controller上需要定义mapping 路径
2. controller的方法有定义路径 /{id}

例如

~~~

@Controller
@ExposesResourceFor(Order.class)
@RequestMapping("/orders")
class OrderController {

  @RequestMapping
  ResponseEntity orders(…) { … }

  @RequestMapping("/{id}")
  ResponseEntity order(@PathVariable("id") … ) { … }
}

~~~

按一下方式使用

~~~

@Controller
class PaymentController {

  @Autowired EntityLinks entityLinks;

  @RequestMapping(…, method = HttpMethod.PUT)
  ResponseEntity payment(@PathVariable Long orderId) {

    Link link = entityLinks.linkToSingleResource(Order.class, orderId);
    …
  }
}
~~~







### 参考资料

[Spring HATEOAS - Reference Documentation](http://docs.spring.io/spring-hateoas/docs/0.20.0.RELEASE/reference/html/)

[HAL - Hypertext Application Language](http://stateless.co/hal_specification.html)
