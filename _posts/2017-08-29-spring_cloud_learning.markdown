---
layout:     post
title:      "Spring Cloud入门学习"
subtitle:   "Spring Cloud微服务框架入门学习笔记"
date:       2017-08-29 09:24:00
author:     "Kei Wu"
header-img: "img/post-bg-01.jpg"
---

### 前言
前段时间趁着空闲时间学习并搭建使用了一下Spring Cloud微服务框架，现在来简单总结一下，做一下学习笔记。

### 服务中心（eureka server）
用IDEA可以快速搭建Spring Boot工程，并在pom.xml中加入cloud的依赖：(Dalston版本，使用Eureka作为服务发现中心)
{% highlight xml %}
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.6.RELEASE</version>
    <relativePath/>
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-eureka-server</artifactId>
    </dependency>
</dependencies>
<dependencyManagement>
    <dependencies>
        <dependency>
           <groupId>org.springframework.cloud</groupId>
           <artifactId>spring-cloud-dependencies</artifactId>
           <version>Dalston.SR1</version>
           <type>pom</type>
           <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
{% endhighlight %}
通过在main入口加入@EnableEurekaServer注解，启动服务中心提供其他服务注册。不过需要注意的一点是服务中心也会尝试将自己作为客户端来注册自己，所以需要配置禁用客户端注册功能。
{% highlight ymal %}
server:
  port: 1111
spring:
  profiles: server1
eureka:
  instance:
    hostname: server1
  client:
    register-with-eureka: false
    fetch-registry: false
    serviceUrl:
      defaultZone: http://127.0.0.1:1112/eureka/
{% endhighlight %}
配置使用ymal语言，指定了server端口和域名（server1需要配置hosts），关闭客户端注册功能。访问httP://localhost:1111/（或者http://server1:1111/）可依访问到服务后台。

### 服务提供者（service provider）
同样创建Spring Boot工程，并加入spring-cloud-starter-eureka依赖（详细查看代码）。
添加@EnableDiscoveryClient注解，通过配置把提供者作为客户端注册到服务中心
{% highlight ymal %}
spring:
  application:
    name: service
server:
  port: 3333
eureka:
  instance:
    hostname: service
  client:
    serviceUrl:
      defaultZone: http://127.0.0.1:1111/eureka/,http://127.0.0.1:1112/eureka/
{% endhighlight %}
代码配置了两个defaultZone，是因为启动了两个eureka server作为高可用服务中心，因此要把服务都注册上去。（实际上只需要配置其中一个，注册中心就能够相互发现而同步到service，配置多个是防止中心宕机等而导致注册失败的情况）启动工程后，访问注册中心后台可以看到提供者已经被注册上去。
因为服务之间是通过http传输的，所以service提供者也是提供http服务，所以只需要配置rest服务即可实现服务间的请求通讯。

### 服务消费者（comsumer）

