---
layout: post
title: "spring整合JMS之ActiveMQ"
date: 2016-05-08
description: "讲解spring整合JMS与ActiveMQ的是基本使用"
tags: spring
---

## 前言

MQ（Message Queue）中文名称消息总线，下文统一简称 MQ 。对于分布式系统来说 MQ 基本上是必不可少的，因为它可以解耦和提高系统的吞吐量。

举个例子：分布式商城系统中的运营商系统，对各个服务的调用关系最多，用到了商家商品服务、内容服务、搜索服务。这种模块之间的依赖也称之为耦合。而耦合越多，之后的维护工作就越困难。那么如何改善系统模块调用关系、减少模块之间的耦合、**提高系统的吐吞量**？其中的一种解决方案----消息中间件。这里也是 MQ 思想。

本篇文章基于我自己的真实开发经验以及对 MQ 的理解，加上工作中使用 ActiveMQ 比较多，所以详细的记录了ActiveMQ 的使用以及与Spring 的整合，除了作为一个记录之外，也希望能够对刚刚学习 ActiveMQ 的程序员有些许帮助。后续会补充 springboot 整合 JMS 的文章。

### 本篇主要从下面几点来分开阐述：

1. ActiveMQ、消息中间件、MQ 之间的关系。
2. JMS 简介
3. ActiveMQ 的安装
4. ActiveMQ 的简单上手
5. Spring 整合 ActiveMQ
6. spirng-jms 中的事务及死信队列

## ActiveMQ、消息中间件、MQ 之间的关系

那么这里提到了几个概念希望大家能够有所明确：MQ，消息中间件，ActiveMQ。那么这三者到底是什么关系呢？下面简单的说明一下应该都能明白了。

为了能够比较好的理解这三个东西的关系，这里和 ORM 、JPA、 HIbernate 作一个对比。大家都知道 ORM 和 JPA 和 Hibernate之间的关系，简单来说就是类似于这种关系。

ORM 是一种思想，JPA 是一种规范、而Hibernate 是对 JPA 的具体实现。

## JMS 简介

#### 什么是 JMS？

JMS（[**Java** ](http://lib.csdn.net/base/java)MessagingService）是Java平台上有关面向消息中间件的技术规范，它便于消息系统中的Java应用程序进行消息交换,并且通过提供标准的产生、发送、接收消息的接口简化企业应用的开发。

   JMS本身只定义了一系列的接口规范，是一种与厂商无关的 API，用来访问消息收发系统。它类似于 JDBC([**java** ](http://lib.csdn.net/base/java)DatabaseConnectivity)：这里，JDBC 是可以用来访问许多不同关系[**数据库**](http://lib.csdn.net/base/mysql)的 API，而 JMS 则提供同样与厂商无关的访问方法，以访问消息收发服务。许多厂商目前都支持 JMS，包括 IBM 的 MQSeries、BEA的 Weblogic JMS service和 Progress 的 SonicMQ，这只是几个例子。 JMS 使您能够通过消息收发服务（有时称为消息中介程序或路由器）从一个 JMS 客户机向另一个 JML 客户机发送消息。消息是 JMS 中的一种类型对象，由两部分组成：报头和消息主体。报头由路由信息以及有关该消息的元数据组成。消息主体则携带着应用程序的数据或有效负载。  

常见的消息中间件产品:  

（1）**ActiveMQ**  

ActiveMQ 是Apache出品，最流行的，能力强劲的开源消息总线。ActiveMQ 是一个完全支持JMS1.1和J2EE 1.4规范的 JMS Provider实现。我们在本次课程中介绍 ActiveMQ的使用。【5W/s吞吐量】  

（2）**RabbitMQ ** 

AMQP协议的领导实现，支持多种场景。淘宝的MySQL集群内部有使用它进行通讯，OpenStack开源云平台的通信组件，最先在金融行业得到运用。【5W/s吞吐量】  

（3）**RocketMQ ** 

阿里开源分布式消息队列系统 【10W/s吞吐量】  

（4）**Kafka**  

Apache下的一个子项目 。特点：高吞吐，在一台普通的服务器上既可以达到16W/s的吞吐量；完全的分布式系统。适合处理海量数据。  

#### JMS 消息正文格式

JMS 定义了五种不同的消息正文格式，以及调用的消息类型，允许你发送并接收以一些不同形式的数据，提供现有消息格式的一些级别的兼容性。

TextMessage -- 字符串消息

MapMessage -- 键值对封装消息

ObjectMessage -- 序列化对象消息

BytesMessage -- 字节封装消息

StreamMessage -- 数据流封装消息

#### JMS消息传递类型

对于消息的传递有两种类型：

一种是点对点的，即一个生产者和一个消费者一一对应：  ![img](/images/post/activemq/clip_image002-1530431583747.gif)  

另一种是发布/ 订阅模式，即一个生产者产生消息并进行发送后，可以由多个消费者进行接收：  

![img](/images/post/activemq/clip_image002-1530431621911.gif)  

## ActiveMQ下载与安装

官方网站下载：http://activemq.apache.org/。 本文使用的是 ActiveMQ-5.14.5 版本。

安装的话和 Tomcat 非常像解压即用，下载下来之后如果是在windows 操作系统的话，根据自己的系统是32位还是64 位的，选择bin目录下不同的 win 目录，比如我的是 64位的，那么直接打开 `bin/win64/activemq.bat` 就可以启动了，默认端口是8161。

Linux 上面的安装是直接解压，然后进入到 bin 里面的目录使用命令 `./activemq start` 就可以启动了。

打开之后的话会弹出如下界面：  

![1530433149782](/images/post/activemq/1530433149782.png)  

点击图中所示位置， Manage ActiveMQ broker 这时候会让你输入用户名和密码。这里默认的账号和密码都是 admin。后期也可以根据自己的需要修改，修改的地方在`conf/users.properties` 这个文件，默认下面是一个 `admin=admin` 所以这就是登录的用户名和密码了。

最后还有一点需要强调的是，不要放到有中文路径的地方，不然会无法启动的。登陆后的页面如下：  

![1530433230465](/images/post/activemq/1530433230465.png)

上面显示的是 ActiveMQ 的管理界面，可以点击图中的 Queue 来查看和管理消息。

点击上方菜单栏的 Queues 可以调整到消息队列的查看和管理界面：  

![1530433411680](/images/post/activemq/1530433411680.png)  

**Number Of Pending Messages****：等待消费的消息 这个是当前未出队列的数量。

**Number Of Consumers****：消费者 这个是消费者端的消费者数量

**Messages Enqueued****：进入队列的消息  进入队列的总数量,包括出队列的。

**Messages Dequeued****：出了队列的消息  可以理解为消费掉的消息总数量。

## ActiveMQ 的简单上手（队列模式）

### 1. 引入依赖

这里使用的是 maven 构建项目，引入依赖： 

```xml
<dependency>
	<groupId>org.apache.activemq</groupId>
	<artifactId>activemq-client</artifactId>
	<version>5.14.5</version>
</dependency>
```

### 2. 创建消息生产者

使用代码操作时的TCP 端口默认为 61616，一般情况下不需要修改。代码如下，关于API 的解释注释中应该写的比较详细了：  

```java
package top.itmap.producer;

import org.apache.activemq.ActiveMQConnectionFactory;

import javax.jms.*;

/** 消息生产者(点对点) */
public class ProducerTest {

    public static void main(String[] args) throws Exception{
        /** 定义连接消息中间件的地址(tcp协议) */
        String brokerURL = "tcp://192.168.126.166:61616";
        /** 创建连接工厂 */
        ConnectionFactory connectionFactory =
                new ActiveMQConnectionFactory(brokerURL);
        /** 创建连接对象 */
        Connection connection = connectionFactory.createConnection();
        /** 开始连接 */
        connection.start();
        /**
         * 创建会话对象
         * 第一个参数transacted: 是否开启事务 true开启 false不开启
         * 第二个参数acknowledgeMode：应答模式
         * Session.AUTO_ACKNOWLEDGE: 自动应答
         * Session.CLIENT_ACKNOWLEDGE: 客户端应答
         * Session.DUPS_OK_ACKNOWLEDGE: 重复确认应答
         * Session.SESSION_TRANSACTED: 会话事务
         * */
        Session session = connection.createSession(false,Session.AUTO_ACKNOWLEDGE);
        /** 创建消息的目标(模式)队列 */
        Destination queue = session.createQueue("test-queue");
        /** 创建消息生产者 */
        MessageProducer producer = session.createProducer(queue);
        /** 创建文本消息 */
        TextMessage tm = session.createTextMessage();
        /** 设置消息内容 */
        tm.setText("您好，JMS我来了！");
        /** 发送消息到消息中间件 */
        producer.send(tm);
        System.out.println("==【生产者】已发送消息==");
        /** 关闭消息生产者、连接、会话 */
        producer.close();
        connection.close();
        session.close();
    }
}

```

点击运行后控制台输出如下代码：  

![1530434454198](/images/post/activemq/1530434454198.png)  

上面的几行警告不需要管，那是日志框架缺少实现导致的这里不需要用到。

查看ActiveMQ 的管理界面会发现多了一行数据  

![1530435034523](/images/post/activemq/1530435034523.png)    

这就是我们刚刚放进去的数据了。  

这条数据还没有消费者消费。下面我们来上代码，来一个消费者

### 3. 创建消息消费者

代码：  

```java
package top.itmap.consumer;

import org.apache.activemq.ActiveMQConnectionFactory;

import javax.jms.*;

/** 消息消费者(点对点) */
public class ConsumerTest {
    public static void main(String[] args) throws Exception{
        /** 定义连接消息中间件的地址(tcp协议) */
        String brokerURL = "tcp://192.168.126.166:61616";
        /** 创建连接工厂 */
        ConnectionFactory connectionFactory = new
                ActiveMQConnectionFactory(brokerURL);
        /** 创建连接对象 */
        Connection connection = connectionFactory.createConnection();
        /** 开始连接 */
        connection.start();
        /**
         * 创建会话对象
         * 第一个参数transacted: 是否开启事务 true开启 false不开启
         * 第二个参数acknowledgeMode：应答模式
         * Session.AUTO_ACKNOWLEDGE: 自动应答
         * Session.CLIENT_ACKNOWLEDGE: 客户端应答
         * Session.DUPS_OK_ACKNOWLEDGE: 重复确认应答
         * Session.SESSION_TRANSACTED: 会话事务
         **/
        Session session = connection.createSession(false,Session.AUTO_ACKNOWLEDGE);
        // 创建消息的目标(模式)队列
        Destination queue = session.createQueue("test-queue");
        // 创建消息消费者
        MessageConsumer consumer = session.createConsumer(queue);
        // 设置消息监听器
        consumer.setMessageListener(new MessageListener() {
            @Override
            public void onMessage(Message message) {
                // 判断消息的类型
                if (message instanceof TextMessage){
                    // 强制转换
                    TextMessage tm = (TextMessage)message;
                    // 获取消息内容
                    try {
                        System.out.println(tm.getText());
                    } catch (JMSException e) {
                        e.printStackTrace();
                    }
                }
            }
        });
    }
}

```

控制台输出：  

![1530435280608](/images/post/activemq/1530435280608.png)  

管理界面：  

![1530435367372](/images/post/activemq/1530435367372.png)  

这里从 1  变成了 0，而且刚刚的消息我们在控制台输出了，这里就代表消息消费成功。

### 发布订阅模式

还有一个是发布订阅的模式，和上面的代码基本上是一样的。只是需要把  

```java
// 创建消息的目标(模式)队列
Destination queue = session.createQueue("test-queue");
```

其中的 `createQueue` 改为`createTopic` 就可以了，这里不再测试。这里值得注意的是，发布订阅模式必须要有至少一个订阅者在消息发布的时候接受到消息，否则该条消息就会丢失。而队列则不是，可以先放到队列中，等有消费者了再消费。

### 自动负载均衡

在上面的案例中，同时开启2个以上的消费者，再次运行生产者，观察每个消费者控制台的输出，会发现只有一个消费者会接收到消息。queue实现了负载均衡，一个消息只能被一个消费者接受，当没有消费者可用时，这个消息会被保存直到有 一个可用的消费者，一个queue可以有很多消费者，他们之间实现了负载均衡， 
所以**Queue实现了一个可靠的负载均衡**。   

topic实现了发布和订阅，当你发布一个消息，所有订阅这个topic的服务都能得到这个消息，所以从1到N个订阅者都能得到一个消息的拷贝， 
只有在消息代理收到消息时有一个有效订阅时的订阅者才能得到这个消息的拷贝。

## spring 整合 JMS

### 配置依赖

在上面的代码的基础上再加上一个依赖，spring 整合JMS 包  

```xml
<!-- spring-jms -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jms</artifactId>
    <version>4.3.10.RELEASE</version>
</dependency>
```

### spring配置文件（队列模式）

说到spring 整合JMS 当然是少不了配置文件的，配置文件 `applicationContext-queue.xml` 如下 ：  

```xml
<?xml version="1.0" encoding="utf-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:jms="http://www.springframework.org/schema/jms"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/jms
       http://www.springframework.org/schema/jms/spring-jms.xsd">
    <!--########### 通用配置 #############-->
    <bean id="activeMQConnectionFactory"
          class="org.apache.activemq.ActiveMQConnectionFactory">
        <!-- 设置brokerURL(连接消息中间件的地址) -->
        <property name="brokerURL" value="tcp://192.168.126.166:61616"/>
    </bean>
    <!-- 配置Spring-JMS的单例连接工厂 -->
    <bean id="connectionFactory"
          class="org.springframework.jms.connection.SingleConnectionFactory">
        <!-- 设置ActiveMQ的连接工厂交由它管理-->
        <property name="targetConnectionFactory" ref="activeMQConnectionFactory"/>
    </bean>

    <!--########### 消息生产者配置 #############-->
    <!-- 配置JmsTemplate模版对象发送消息 -->
    <bean id="jmsTemplate" class="org.springframework.jms.core.JmsTemplate">
        <!-- 设置连接工厂 -->
        <property name="connectionFactory" ref="connectionFactory"/>
        <!-- 设置默认的目标 -->
        <property name="defaultDestination" ref="queue"/>
    </bean>
    <!-- 配置消息的目标(点对点) -->
    <bean id="queue" class="org.apache.activemq.command.ActiveMQQueue">
        <!-- 设置队列名称 -->
        <constructor-arg value="spring-queue"/>
    </bean>

    <!--########### 消息消费者配置 #############-->
    <!--
        配置监听器容器
        connection-factory: 连接工厂
        destination-type: 目标类型
    -->
    <jms:listener-container connection-factory="connectionFactory"
                            destination-type="queue">
        <!-- 配置监听器 destination: 队列名称 ref: 引用消息监听器Bean -->
        <jms:listener destination="spring-queue" ref="queueMessageListener"/>
    </jms:listener-container>
    <!-- 配置消息监听器Bean -->
    <bean id="queueMessageListener"
          class="top.itmap.activemq.spring.listener.QueueMessageListener"/>
</beans>

```

配置文件解释： 将JMS 的连接工厂交给 spring 管理。以及配置消息中间件的连接地址，还有配置生产者与消费者。

###  编写生产者代码（队列模式）：

```xml
package top.itmap.activemq.spring.producer;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.springframework.jms.core.JmsTemplate;
import org.springframework.jms.core.MessageCreator;

import javax.jms.JMSException;
import javax.jms.Message;
import javax.jms.Session;
import javax.jms.TextMessage;

/**
 * Description: queue test
 *
 * Author: AllenLei (leicong@foxmail.com)
 * 
 */
public class QueueTest {
    public static void main(String[] args){
        /** 创建Spring容器 */
        ApplicationContext ac = new
                ClassPathXmlApplicationContext("applicationContext-queue.xml");
        /** 获取JmsTemplate */
        JmsTemplate jmsTemplate = ac.getBean(JmsTemplate.class);
        /** 发送消息 */
        jmsTemplate.send(new MessageCreator() {
            @Override
            public Message createMessage(Session session)
                    throws JMSException {
                TextMessage tm = session.createTextMessage();
                tm.setText("您好，leicong！");
                return tm;
            }
        });
    }
}
```

### 编写消息消费者代码（队列模式）

```java
package top.itmap.activemq.spring.listener;

import org.springframework.jms.listener.SessionAwareMessageListener;

import javax.jms.JMSException;
import javax.jms.Session;
import javax.jms.TextMessage;

/** 队列消息监听器类 */
public class QueueMessageListener implements
        SessionAwareMessageListener<TextMessage> {
    @Override
    public void onMessage(TextMessage message, Session session) throws JMSException {
        System.out.println("======QueueMessageListener=====");
        System.out.println(message.getText());
    }
}

```

### 控制台输出

准备好以上代码后，直接运行main 方法。控制台出现如下代码：  

![1530437346714](/images/post/activemq/1530437346714.png)  

这里由于消息的监听器交给 spring 管理了。那么只要 spring 容器启动，则默认会启动监听器来监听消息。所以直接消费了消息队列。

### spring 配置文件（主题模式）

由于spring 整合JMS 的主题模式和队列模式编码基本一样只是配置文件不同而已，这里只是放出 spring 整合 JMS 主题模式的配置文件，测试的话请自行测试。

```xml
<?xml version="1.0" encoding="utf-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:jms="http://www.springframework.org/schema/jms"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/jms
       http://www.springframework.org/schema/jms/spring-jms.xsd">

    <!--########### 通用配置 #############-->
    <bean id="activeMQConnectionFactory"
          class="org.apache.activemq.ActiveMQConnectionFactory">
        <!-- 设置brokerURL(连接消息中间件的地址) -->
        <property name="brokerURL" value="tcp://192.168.126.166:61616"/>
    </bean>
    <!-- 配置Spring-JMS的单例连接工厂 -->
    <bean id="connectionFactory"    class="org.springframework.jms.connection.SingleConnectionFactory">
        <!-- 设置ActiveMQ的连接工厂交由它管理-->
        <property name="targetConnectionFactory" ref="activeMQConnectionFactory"/>
    </bean>

    <!--########### 消息生产者配置 #############-->
    <!-- 配置JmsTemplate模版对象发送消息 -->
    <bean id="jmsTemplate" class="org.springframework.jms.core.JmsTemplate">
        <!-- 设置连接工厂 -->
        <property name="connectionFactory" ref="connectionFactory"/>
        <!-- 设置默认的目标 -->
        <property name="defaultDestination" ref="topic"/>
    </bean>
    <!-- 配置消息的目标(发布与订阅) -->
    <bean id="topic" class="org.apache.activemq.command.ActiveMQTopic">
        <!-- 设置主题名称 -->
        <constructor-arg value="spring-topic"/>
    </bean>

    <!--########### 消息消费者配置 #############-->
    <!--
        配置监听器容器
        connection-factory: 连接工厂
        destination-type: 目标类型
    -->
    <jms:listener-container connection-factory="connectionFactory"
                            destination-type="topic">
        <!-- 配置监听器 destination: 主题名称 ref: 引用消息监听器Bean -->
        <jms:listener destination="spring-topic" ref="topicMessageListener"/>
    </jms:listener-container>
    <!-- 配置消息监听器Bean -->
    <bean id="topicMessageListener"
class="top.itmap.activemq.spring.listener.TopicMessageListener"/>
</beans>

```

### 注意事项：

以下是一些注意事项需要说明的： 

1. 配置监听器类的时候，需要实现的接口是 spring 的  `SessionAwareMessageListener<TextMessage>` 而不是原生的 JMS 提供的 MessageListener 这个接口了。spring 已经对齐做了封装而且简化了操作，不需要在获取消息的时候对消息的类型进行强转，而是直接可以在接口中指定消息的类型。
2. 由于考虑到部署的时候需要修改配置文件中的 brokerURL，最好是将这个地址配置在另外的文件中，如 `jms-properties` 。这样易于部署的时候修改。在部署上去的时候需要注意spring 配置文件加载的先后顺序，如果说有多个spring 的配置文件的话，那么，spring 加载 `jms-properties` 配置文件的时间不能晚于 spring 整合 jms 的配置的时间。否则会发生加载不到 brokerURL 的情况。

### ActiveMQ中的事务

在实际情况中 spring 整合 JMS 时需要考虑到事务的问题，因为如果消息的消费途中产生异常的话，那么理应回滚此次操作。而如果不配置事务的话则不论消息的消费是否完成，都会返回消息消费成功。所以事务的配置这里也提一下。配置好事务之后，需要在最后做业务处理时使用 `session.commit()` 和 `session.rollback()` 这个两个方法控制事务是否提交。这里需要在配置文件中配置。在配置消息监听器的时候指定应答模式为`acknowlege="transacted"`。 如下所示：  

```xml
<!-- 设置应答模式为事务，开启事务 --> 
<jms:listener-container connection-factory="connectionFactory"
                            destination-type="topic" acknowlege="transacted" >
```

这样配置的话就需要在消费消息时提交事务：  

```java

/** 队列消息监听器类 */
public class QueueMessageListener implements
        SessionAwareMessageListener<TextMessage> {
    @Override
    public void onMessage(TextMessage message, Session session) throws JMSException {
        System.out.println("======QueueMessageListener=====");
        try{
             System.out.println(message.getText());
            // 消费消息、执行业务代码
            // .....
            // 代码没有异常的话提交事务
            session.commit();
        } catch (Exception e) {
            // 代码出现异常，回滚事务
            session.rollback();
            // 异常转型
            throw new RuntimeException(e);
        }
    }
}

```

如果能够控制好ActiveMQ 中的事务的话那么对于 ActiveMQ 的使用基本上算是到位了。那么如果执行代码出现了异常的话那消息走哪里去了呢？接下来就得提一提 ActiveMQ 的死信队列了。

#### ActiveMQ 中死信队列DLQ（Dead Letter Queue）

死信队列的意思是当 ActiveMQ 的消费者执行业务时出现异常的话，那么ActiveMQ 默认会自动重复的发送 6 此消息，确认是否能被消费到。如果 7 次消息（本身一次，加上rollback 后的 6 次）都没有被正确的消费到的话，那么该次消息就会被放到一个叫做死信队列的队列中去存放着，等待处理。

这里使用 `int i = 1/0;` 简单的模拟异常后，死信队列出现时的ActiveMQ 管理界面图：  

![1530443844424](/images/post/activemq/1530443844424.png)  

这就是 ActiveMQ 的死信队列了。

## 总结

本篇文章比较详细的介绍了 ActiveMQ 常用方法与事务的控制，一般情况下掌握这么多足够应付大部分情况了。在分布式系统中，灵活的使用队列可以解耦和提高系统的吞吐量。将耗时的业务流程放到后台去异步执行，有效的提升了用户体验，将之前的同步变成了现在的异步处理使得系统显得更流畅。当然也不是所有的业务都使用队列去处理，这样反而会起到反效果，如果说比较简单的业务的话完全没必要再多增加一层队列层，直接执行业务代码会比较好。

下面简单说下我在电商项目中使用到队列的模块，仅供参考。  

1. 商品详情页面网页静态化。
2. 索引库的 CRUD。
3. 邮件短信的发送。

等等，这里就不再罗列，只是提供一个思路以供借鉴。



<br/>

**原创声明**   

转载请注明: [雷聪的博客](https://allenleic.github.io) » [{{ page.title }}](https://allenleic.github.io/2016/05/{{ page.title }})