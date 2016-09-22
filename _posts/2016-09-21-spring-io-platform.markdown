---
layout: post
title:  "Spring Io Platform使用教程"
date:   2016-09-21
categories: spring
---

Spring IO Platform  将各种spring API  结合在一起 , 提供了各个可以一起工作的api版本号。

Spring IO Platform 是在Spring Boot 的基础上构建的，进一步简化了 java开发。在没有特别要求的情况下无需关注依赖版本，能够更加快速安全的构建应用。

### 使用 Spring IO Platform
Spring IO Platform 支持 maven 以及 Gradle  ，因为不了解 Gradle  这里不做说明了。

下面是 maven 的引入方法

~~~
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>io.spring.platform</groupId>
            <artifactId>platform-bom</artifactId>
            <version>2.0.7.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
~~~

或者以 父模块的方式引入

~~~
<parent>
    <groupId>io.spring.platform</groupId>
    <artifactId>platform-bom</artifactId>
    <version>2.0.7.RELEASE</version>
    <relativePath/>
</parent>
~~~

### 查看依赖版本
通过查看 platform-bom-2.0.7.RELEASE.pom 内容获取依赖版本。

### 重写依赖版本
如果需要修改特定的jar版本 在maven可以使用 <properties> 节点。

~~~
<properties>
    <foo.version>1.1.0.RELEASE</foo.version>
</properties>
~~~


因为spring-boot 的依赖是以父模块的方式导入至  io.spring.platform,因此不能采用<properties> 节点,重写版本号。

目前 io.spring.platform 版本号是2.0.7.RELEASE，
其依赖的spring-boot 是1.3.7 如果要使用更高版本的spring-boot，

在采用 父模块的情况下 需要依次引入带有版本号的依赖

~~~
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <version>1.4.0.RELEASE</version>
    </dependency>
</dependencies>
~~~

在采用依赖dependencyManagement 引入的情况下， 可以在 io.spring.platform之前先引入 spring-boot-starter-parent （在其后引入不产生作用）

~~~
<dependencyManagement>
   <dependencies>

       <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-parent</artifactId>
           <version>1.4.0.RELEASE</version>
           <type>pom</type>
           <scope>import</scope>
       </dependency>

       <dependency>
           <groupId>io.spring.platform</groupId>
           <artifactId>platform-bom</artifactId>
           <version>2.0.7.RELEASE</version>
           <type>pom</type>
           <scope>import</scope>
       </dependency>

   </dependencies>
</dependencyManagement>
~~~
