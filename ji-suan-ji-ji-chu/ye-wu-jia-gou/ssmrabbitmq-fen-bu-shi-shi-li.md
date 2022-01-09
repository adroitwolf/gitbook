---
title: ssm+rabbitmq 分布式实例
date: 2019-04-15T18:39:54.000Z
categories:
  - j2EE
tags:
  - rabbitmq
  - ssm
---

# 在分布式架构下利用rabbitmq完成消息队列

***

## 配置文件详解

***

> 我们利用rabbit:admin直接代码绑定交换机和队列,这里是开发常用的方法 最后我们会利用web端界面实现消息的分发实例

### 生产者

***

#### 配置文件详解

***

* 下面是rabbitmq的基础设置

```xml
 <!-- 定义连接工厂 -->
<rabbit:connection-factory id="connectionFactory"
                               username="${mq.username}" password="${mq.password}" host="${mq.host}" port="${mq.port}"
                               virtual-host="${mq.vh}" />

    <!-- 定义rabbit template 用于数据的接收和发送 -->
    <rabbit:template id="amqpTemplate" connection-factory="connectionFactory"
                     exchange="solrExChange"></rabbit:template>
```

* 定义队列

```xml
 <!--定义queue  说明：durable:是否持久化 exclusive: 仅创建者可以使用的私有队列，断开后自动删除 auto_delete: 当所有消费客户端连接断开后，是否自动删除队列-->
    <rabbit:queue name="chase1" durable="true" auto-delete="false" exclusive="false" />
```

* 定义交换机，这里我们定义topic模式,这里我们绑定队列

```xml
 <!--topic 模式：发送端不是按固定的routing key发送消息，而是按字符串“匹配”发送，接收端同样如此。 -->
<!--durable : 是否持久化 auto-delete：是否自动删除  -->
    <rabbit:topic-exchange name="solrExChange"
                           durable="true" auto-delete="false">
         <rabbit:bindings>
             <!-- 这个pattern是topic特有的通配符模式 -->
            <rabbit:binding queue="chase1" pattern="item.#"></rabbit:binding>
        </rabbit:bindings>
    </rabbit:topic-exchange>
```

* 利用代码同步队列和交换机

```xml
<!-- 通过指定下面的admin信息，当前productor中的exchange和queue会在rabbitmq服务器上自动生成 -->
    <rabbit:admin id="amqpAdmin" connection-factory="connectionFactory" />
```

> 注意: 如果这里不写rabbit:admin,也就是说我们利用rabbitmq的web界面手动绑定的话,我们并不需要在交换机上绑定队列和定义队列

* 完成的配置文件
  * applicationContext-rabbit.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:rabbit="http://www.springframework.org/schema/rabbit"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
    http://www.springframework.org/schema/rabbit
    http://www.springframework.org/schema/rabbit/spring-rabbit-1.0.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <!-- 引入外部数据库的配置文件，location位置填写的是相对位置 -->
    <context:property-placeholder location="classpath:properties/rabbit.properties" ignore-unresolvable="true"/>


    <rabbit:connection-factory id="connectionFactory"
    username="${mq.username}" password="${mq.password}" host="${mq.host}" port="${mq.port}"
    virtual-host="${mq.vh}" />

    <!-- 定义rabbit template 用于数据的接收和发送 -->
    <rabbit:template id="amqpTemplate" connection-factory="connectionFactory"
    exchange="solrExChange"></rabbit:template>

    <!--定义queue  说明：durable:是否持久化 exclusive: 仅创建者可以使用的私有队列，断开后自动删除 auto_delete: 当所有消费客户端连接断开后，是否自动删除队列-->
    <rabbit:queue name="chase1" durable="true" auto-delete="false" exclusive="false" />

    <!--topic 模式：发送端不是按固定的routing key发送消息，而是按字符串“匹配”发送，接收端同样如此。 -->
    <rabbit:topic-exchange name="solrExChange"
    durable="false" auto-delete="false">
        <rabbit:bindings>
            <rabbit:binding queue="chase1" pattern="item.#"></rabbit:binding>
        </rabbit:bindings>
    </rabbit:topic-exchange>

    <!-- 通过指定下面的admin信息，当前productor中的exchange和queue会在rabbitmq服务器上自动生成 -->
    <rabbit:admin id="amqpAdmin" connection-factory="connectionFactory" />

</beans>
```

#### 具体代码

***

```java
package com.rabbit.producer.service;

import org.springframework.amqp.core.AmqpTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import javax.annotation.Resources;

/**
 * Created with IntelliJ IDEA.
 * User: WHOAMI
 * Date: 2019/4/15
 * Time: 8:46
 * Description: :/TODO_
 */
@Service
public class RabbitService  {
	//注入模板
    @Autowired
    private AmqpTemplate amqpTemplate;

    public void SendTest(){
        System.out.println("开始发送消息");
        //这里是代码核心处
       amqpTemplate.convertAndSend("item.message","发送消息");
    }

}
```

### 消费者

***

#### 配置文件详解

***

* 下面是rabbitmq的基础设置

```xml
 <!-- 定义连接工厂 -->
<rabbit:connection-factory id="connectionFactory"
                               username="${mq.username}" password="${mq.password}" host="${mq.host}" port="${mq.port}"
                               virtual-host="${mq.vh}" />

    <!-- 定义rabbit template 用于数据的接收和发送 -->
    <rabbit:template id="amqpTemplate" connection-factory="connectionFactory"
                     exchange="solrExChange"></rabbit:template>
```

* 定义队列

```xml
 <!--定义queue  说明：durable:是否持久化 exclusive: 仅创建者可以使用的私有队列，断开后自动删除 auto_delete: 当所有消费客户端连接断开后，是否自动删除队列-->
    <rabbit:queue name="chase1" durable="true" auto-delete="false" exclusive="false" />
```

* 利用代码同步队列和交换机

```xml
<!-- 通过指定下面的admin信息，当前productor中的exchange和queue会在rabbitmq服务器上自动生成 -->
    <rabbit:admin id="amqpAdmin" connection-factory="connectionFactory" />
```

* 定义消费者

```xml
 <!-- 消息接收者 具体到特定类 -->
    <bean id="Consumer" class="com.rabbit.consumer.utils.Consumer"></bean>
```

* 定义监听器

```xml
<!-- queue litener 观察 监听模式 当有消息到达时会通知监听在对应的队列上的监听对象
    acknowledeg = "manual"，意为表示该消费者的ack方式为手动 默认为auto-->
    <rabbit:listener-container connection-factory="connectionFactory" acknowledeg = "manual">
        <rabbit:listener  queues="chase1"  ref="Consumer"/>
    </rabbit:listener-container>
```

* 完成配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:rabbit="http://www.springframework.org/schema/rabbit"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
    http://www.springframework.org/schema/rabbit
    http://www.springframework.org/schema/rabbit/spring-rabbit-1.0.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <!-- 引入外部数据库的配置文件，location位置填写的是相对位置 -->
    <context:property-placeholder location="classpath:properties/rabbit.properties" ignore-unresolvable="true"/>


    <rabbit:connection-factory id="connectionFactory"
    username="${mq.username}" password="${mq.password}" host="${mq.host}" port="${mq.port}"
    virtual-host="${mq.vh}" />

    <!--定义queue  说明：durable:是否持久化 exclusive: 仅创建者可以使用的私有队列，断开后自动删除 auto_delete: 当所有消费客户端连接断开后，是否自动删除队列-->
    <rabbit:queue name="chase1" durable="true" auto-delete="false" exclusive="false" />

    <!-- 消息接收者 -->
    <bean id="Consumer" class="com.rabbit.consumer.utils.Consumer"></bean>


    <!-- queue litener 观察 监听模式 当有消息到达时会通知监听在对应的队列上的监听对象
    acknowledeg = "manual"，意为表示该消费者的ack方式为手动-->
    <rabbit:listener-container connection-factory="connectionFactory" acknowledge="manual">
        <rabbit:listener  queues="chase1"  ref="Consumer"/>
    </rabbit:listener-container>

    <!-- 通过指定下面的admin信息，当前productor中的exchange和queue会在rabbitmq服务器上自动生成 -->
    <rabbit:admin connection-factory="connectionFactory" />

</beans>
```

#### 具体代码

***

我们要定义消费者，那么我们需要实现ChannelAwareMessageListener 和 MessageListener接口

* 两个接口源码如下

```java

public interface MessageListener {
 
    void onMessage(Message message);
 
}
 
public interface ChannelAwareMessageListener {
    void onMessage(Message message, Channel channel) throws Exception;
 
}
```

> 个人说明： 这两个接口的区别就是 Channel 会手动进行ack

```java
package com.rabbit.consumer.utils;

import com.rabbitmq.client.Channel;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.rabbit.core.ChannelAwareMessageListener;
import org.springframework.stereotype.Component;

/**
 * Created with IntelliJ IDEA.
 * User: WHOAMI
 * Date: 2019/4/15
 * Time: 9:01
 * Description: :/TODO_
 */
@Component
public class Consumer implements ChannelAwareMessageListener {

    @Override
    public void onMessage(Message message, Channel channel) throws Exception {
        String msg = new String(message.getBody(),"utf-8");
        //消息的标识，false只确认当前一个消息收到，true确认所有consumer获得的消息
        channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
        //ack返回false，并重新回到队列，api里面解释得很清楚
//        channel.basicNack(message.getMessageProperties().getDeliveryTag(), false, true);
        //true拒绝消息 false确认接受到消息
        //channel.basicReject(message.getMessageProperties().getDeliveryTag(), false);
        System.out.println("消费者消费掉了消息:" + msg);
    }
}
```

## 错误笔记

***

注意 ，如果web端里面有配置文件里面声明的交换机或者队列，如果配置文件里面 declare或者auto-delete不相同的时候会爆出500的错误！并且不会绑定数据！

![](https://s2.ax1x.com/2019/04/15/Ajj3Mq.png)

![](https://s2.ax1x.com/2019/04/15/Ajj0zR.png)
