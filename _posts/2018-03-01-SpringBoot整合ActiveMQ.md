---
layout: post
title: "SpringBoot整合ActiveMQ"
date: 2018-03-01
description: "介绍如何使用SpringBoot整合JMS 实现短信发送"
tags: spring-boot
---

## 前言

我在之前已经介绍过了 MQ 在分布式系统中的使用，以及与spring 的整合，本篇文章不再累述，没有看过的可以看一下。[spring整合JMS之ActiveMQ](https://allenleic.github.io/2016/05/spring%E6%95%B4%E5%90%88JMS%E4%B9%8BActiveMQ/)。  

本篇文章主要是介绍如何使用 springboot 整合 JMS 来开发微服务，通过 MQ 传递消息给来完成短信或者邮件等功能的异步处理，将这些功能可以做成微服务。这么做的好处是，可以复用这些组件，是代码解耦，提高系统的容错，提高系统的吞吐量等。比如说：使用springboot 监听 MQ 队列中的消息来发送短信，那么其他的分布式子系统如果需要发送短信的话只需要向 MQ 中发送手机号码和验证码即可，通过这种方式实现异步发送短信，实现高可用与解耦，这是一种非常不错的短信验证码解决方案。

本篇文章的代码使用 maven 构建。

### 本篇文章主要写了

1. SpringBoot 整合 ActiveMQ的依赖说明。
2. 使用内嵌 ActiveMQ。
3. 使用外置 ActiveMQ。

## SpringBoot 整合 JMS 和 ActiveMQ的使用

使用springboot 整合 ActiveMQ 实际上比想象中的要简单很多，这得益于springboot 的理念：**约定优于配置**。好了，废话少说，直接先上依赖代码。

### 1. 添加spring-boot-starter-activemq启动器

pom.xml 文件如下：  

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <!-- 父工程 -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.3.RELEASE</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>spring-boot-activemq</artifactId>
    <dependencies>
        <!-- 添加 springboot 整合ActiveMQ 的启动器 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-activemq</artifactId>
            <version>1.5.6.RELEASE</version>
        </dependency>

        <!-- 添加 springboot 整合 web 启动器-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <version>2.0.3.RELEASE</version>
        </dependency>
    </dependencies>

    <properties>
        <!-- 更改JDK版本 -->
        <java.version>1.8</java.version>
    </properties>

</project>
```

**配置说明**：  使用 springboot  配置非常的方便，这里引入 web 启动器，是为了测试用。

1. springboot 引入web 启动器之后默认内嵌了 Tomcat，不进行配置默认的访问地址是 http://localhost:8080 这里测试使用不作配置。
2. spring-boot-starter-activemq 启动器内嵌了 ActiveMQ 所以可以直接使用。也可以通过配置 broker-url 来使用外部的 MQ。
3. 父工程中整合了大量的配置，一般情况下都是需要的。

### 使用springboot内嵌的ActiveMQ

#### springboot 使用内嵌 ActiveMQ，编写controller 代码如下：

```java
package top.itmap.springboot.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jms.core.JmsTemplate;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * Description: 发送消息队列
 *
 * Author: AllenLei (leicong@foxmail.com)
 * date: 2018/03/02
 */
@RestController
public class ProducerController {

    /* 直接注入即可 */
    @Autowired
    private JmsTemplate jmsTemplate;

    /* 发送短信的方法 */
    @GetMapping("send")
    public String sendMessage(String message) {
        /**
         * 第一个参数为队列名字
         * 第二个参数为携带的信息
         */
        jmsTemplate.convertAndSend("sendMessage", message);
        return "success";
    }

}
```

#### 编写监听器类

```java
package top.itmap.springboot.listener;

import org.springframework.jms.annotation.JmsListener;
import org.springframework.stereotype.Component;

/**
 * Description: 监听测试
 *
 * Author: AllenLei (leicong@foxmail.com)
 * date: 2018/03/02
 */
@Component
public class MyMessageListener {

    /**
     * 直接在方法上添加注解，注解上面配置监听的队列名称
     *
     */
    @JmsListener(destination = "sendMessage")
    public void listener(String message) {
        System.out.println("接收到的消息：" + message);
    }
}

```

#### 启动引导类

```java
package top.itmap.springboot;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

/**
 * Description: 引导
 *
 * Author: AllenLei (leicong@foxmail.com)
 * date: 2018/03/02
 */
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}

```

到此使用内嵌的 ActiveMQ 简单demo 就编写完成了。预期的结果是：启动引导类，访问controller携带参数，之后会在页面看到success，会在控制台输出携带的参数信息

#### 测试结果：

点击引导类启动工程。不需要配置任何东西，直接请求 `http://localhost:8080/send?message=hello-mq` 这个地址，使用get 方法携带了一个数据。  

浏览器：  

![1530521269040](/images/posts/springboot/activemq/1530521269040.png)  

控制台输出：  

![1530521322682](/images/posts/springboot/activemq/1530521322682.png)  

符合预期，使用spirngboot 内嵌的ActiveMQ 队列测试成功。

### 使用外部 ActiveMQ 

一般情况下我们都会使用外部的 ActiveMQ ，那么这时候就需要指定外部 MQ 的地址了。这时候我们编写配置文件 application.yml 如下：  

```yaml
# 配置外部的 ActiveMQ 地址
spring:
  activemq:
    broker-url: tcp://192.168.126.166:61616
# 配置服务端口
server:
  port: 9090
```

这里配置个服务端口可能效果更明显。

#### 编写controller

直接在之前写的`ProducerController` 中添加以下代码，这次我们测试发送 map 集合数据该怎么传递：

```java
/* 使用外部 mq 发送 map 信息 */
@GetMapping("/sendMap")
public void sendMap(){
    Map<String, String> map = new HashMap<>();
    map.put("id", "1");
    map.put("name", "张三");
    jmsTemplate.convertAndSend("sendMessage.map", map);
}
```

####  编写监听器

使用springboot 由于监听 MQ 只需要在方法上面写上注解就可以了，也就是说可以在一个类中监听多个消息队列。

```java
    /**
     * 使用外部 MQ 测试
     * @param map 接收MQ 中传过来的参数
     */
    @JmsListener(destination="sendMessage.map")
    public void readMapMessage(Map<String,String> map){
        // 直接输出信息
        System.out.println(map);
    }
```

#### 测试结果

重启服务后访问`http://localhost:9090/sendMap` 。  

控制台输出  

![1530522330043](/images/posts/springboot/activemq/1530522330043.png)  

登录到外部ActiveMQ 管理器查看：  

![1530522413665](/images/posts/springboot/activemq/1530522413665.png)  

测试成功，刚刚的消息被成功消费了。 到此springboot 整合 ActiveMQ 队列的内嵌和和外置队列都已经实现了。

## 总结：

以上是使用springboot 整合 ActiveMQ 队列模式的简单案例，需要注意的是发送队列消息时 `jmsTemplate.convertAndSend("sendMessage.map", map);` 这里的第二个参数是 object 类型，也就是说参数可以是任意类型。不过消费者需要消费的方法参数上面需和这里传递的保持类型一致。

通过 springboot 整合 ActiveMQ 的方式可以将发送短信或者邮件分别都通过 MQ 来传递消息做成一个服务。这样在分布式系统中可以提升系统的执行效率。



<br/>

**原创声明**   

转载请注明: [雷聪的博客](https://allenleic.github.io) » [{{ page.title }}](https://allenleic.github.io/2018/03/{{ page.title }})