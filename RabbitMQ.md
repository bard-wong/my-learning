# Linux启动RabbitMQ

```
#启动RabbitMQ服务
systemctl start rabbitmq-server
#查看RabbitMQ服务
systemctl status rabbitmq-server.service
#停止服务
systemctl stop rabbitmq-server
#开机启动服务
systemctl enable rabbitmq-server
```

# RabbitMQ管理界面及授权操作

## 1.开启

```
#RabbitMQ管理界面插件开启
rabbitmq-plugins enable rabbitmq_management
```

开启后默认端口`15672`默认账号密码`guest`默认情况只能在localhost本机下访问，所以需要添加一个远程登录的用户

注意：远程服务器上需要开放`15672`端口

## 2.授权账号和密码

新增用户

```
rabbitmqctl add_user admin admin
```

设置用户分配操作权限

```
rabbitmqctl set_user_tags admin administrator
```

用户级别：

1. administrator：可以登录控制台、查看所有信息，可以对RabbitMQ进行管理
2. monitoring：监控者 登录控制台，查看所有信息
3. policymaker：策略制定着 登录控制台，指定策略
4. managment：普通管理员 登录控制台

为用户添加资源权限

```
rabbitmqctl.bat set_permissions -p admin ".*" ".*" ".*"
```



## 3.常用命令小结

```
rabbitmqctl add_user 账号 密码
rabbitmqctl set_user_tags 账号 用户级别
rabbitmqctl change_password 账号 新密码
rabbitmqctl delete_user 账号 #删除用户
rabbitmqctl list_users #查看用户清单
rabbitmqctl.bat set_permissions -p / 账号 ".*" ".*" ".*"
rabbitmqctl.bat set_permissions -p / root ".*" ".*" ".*"
```

# RabbitMQ入门案例 - Simple 简单模式

## 实现步骤

1：jdk1.8
2：构建一个maven工程
3：导入rabbitmq的maven依赖
4：启动rabbitmq-server服务
5：定义生产者
6：定义消费者
7：观察消息的在rabbitmq-server服务中的过程

pom依赖

```xml
    <dependencies>
    	#Java原生依赖
        <dependency>
            <groupId>com.rabbitmq</groupId>
            <artifactId>amqp-client</artifactId>
            <version>5.12.0</version>
        </dependency>
        #spring依赖
        <dependency>
            <groupId>org.springframework.amqp</groupId>
            <artifactId>spring-rabbit</artifactId>
            <version>2.3.6</version>
        </dependency>
        #springboot依赖
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
            <version>2.4.4</version>
        </dependency>
    </dependencies>
```

## Simple队列

### 生产者

```java
public class Producer {
    public static void main(String[] args) {
        //1：创建连接工厂
        ConnectionFactory connectionFactory = new ConnectionFactory();
        //2：设置连接属性
        connectionFactory.setHost("192.168.199.132");
        connectionFactory.setPort(5672);
        connectionFactory.setUsername("admin");
        connectionFactory.setPassword("admin");
        connectionFactory.setVirtualHost("/");
        //定义连接和通道
        Connection connection = null;
        Channel channel = null;

        try {
            //3：通过连接工厂创建连接Connection
            connection = connectionFactory.newConnection("生产者");
            //4：通过连接获取通道Channel
            channel = connection.createChannel();
            //5：通过通道创建交换机，声明队列，绑定关系，路由key，发送消息，和接受消息
            String queueName = "queue1";

            /**
             * @params1 队列的名称
             * @params2 是否持久化durable=false 所谓持久化消息是否存盘，如果false非持久化true是持久化？非持久化会存盘吗？
             * @params3 排他性，是否是独占独立
             * @params4 是否自动删除，随着最后一个消费者消息完毕消息以后是否把队列自动删除
             * @params5 携带的附属参数
             * */
            channel.queueDeclare(queueName,false,false,false,null);
            //6：准备消息内容
            String message = "hello RabbitMQ";
            //7：发送消息给队列queen
            channel.basicPublish("",queueName,null,message.getBytes());
            System.out.println("消息发送成功！！");
        } catch (IOException e) {
            e.printStackTrace();
        } catch (TimeoutException e) {
            e.printStackTrace();
        } finally {
            //8：关闭通道
            if (channel != null && channel.isOpen()) {
                try {
                    channel.close();
                } catch (IOException e) {
                    e.printStackTrace();
                } catch (TimeoutException e) {
                    e.printStackTrace();
                }
            }
            //9：关闭连接
            if (connection != null && connection.isOpen()) {
                try {
                    connection.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }

    }
}
```

### 消费者

```java
public class Consumer {
    public static void main(String[] args) {
        //1：创建连接工厂
        ConnectionFactory connectionFactory = new ConnectionFactory();
        //2：设置连接属性
        connectionFactory.setHost("192.168.199.132");
        connectionFactory.setPort(5672);
        connectionFactory.setUsername("admin");
        connectionFactory.setPassword("admin");
        connectionFactory.setVirtualHost("/");
        //定义连接和通道
        Connection connection = null;
        Channel channel = null;

        try {
            //3：通过连接工厂创建连接Connection
            connection = connectionFactory.newConnection("消费者");
            //4：通过连接获取通道Channel
            channel = connection.createChannel();
            //5：通过通道创建交换机，声明队列，绑定关系，路由key，发送消息，和接受消息
            /**
             * @params1 交换机
             * @params2 队列、路由key
             * @params3 消息的状态控制
             * @params4 消息主题
             * */

            channel.basicConsume("queue1", true, new DeliverCallback() {
                @Override
                public void handle(String consumerTag, Delivery message) throws IOException {
                    System.out.println("收到消息是" + new String(message.getBody(), "UTF-8"));
                }
            }, new CancelCallback() {
                @Override
                public void handle(String consumerTag) throws IOException {
                    System.out.println("接收消息失败！");
                }
            });
            System.out.println("开始接收消息");
            System.in.read();
        } catch (IOException e) {
            e.printStackTrace();
        } catch (TimeoutException e) {
            e.printStackTrace();
        } finally {
            //6：关闭通道
            if (channel != null && channel.isOpen()) {
                try {
                    channel.close();
                } catch (IOException e) {
                    e.printStackTrace();
                } catch (TimeoutException e) {
                    e.printStackTrace();
                }
            }
            //7：关闭连接
            if (connection != null && connection.isOpen()) {
                try {
                    connection.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }

    }
}

```

## AMQP

AMQP全称：Advanced Message Queuing Protocol(高级消息队列协议)。是应用层协议的一个开发标准，为面向消息的中间件设计。

### AMQP生产者流转过程

![](E:\笔记\image\be902d1a8713e221d1f93e2f6fae936c.png)

### AMQP消费者流转过程

![](E:\笔记\image\760304fced101daf71919620f93d51cb.png)

# RabbitMQ的核心组成部分

## RabbitMQ的核心组成部分

![](E:\笔记\image\becf4792d1aafac1858c9ad9222d5676.png)

核心概念：
**Server**：又称Broker ,接受客户端的连接，实现AMQP实体服务。 安装rabbitmq-server
**Connection**：连接，应用程序与Broker的网络连接 TCP/IP/ 三次握手和四次挥手
**Channel**：网络信道，几乎所有的操作都在Channel中进行，Channel是进行消息读写的通道，客户端可以建立对各Channel，每个Channel代表一个会话任务。
**Message** :消息：服务与应用程序之间传送的数据，由Properties和body组成，Properties可是对消息进行修饰，比如消息的优先级，延迟等高级特性，Body则就是消息体的内容。
**Virtual Host**：虚拟地址，用于进行逻辑隔离，最上层的消息路由，一个虚拟主机理由可以有若干个Exhange和Queueu，同一个虚拟主机里面不能有相同名字的Exchange
**Exchange**：交换机，接受消息，根据路由键发送消息到绑定的队列。(不具备消息存储的能力)
**Bindings**：Exchange和Queue之间的虚拟连接，binding中可以保护多个routing key.
**Routing key**：是一个路由规则，虚拟机可以用它来确定如何路由一个特定消息。
**Queue**：队列：也成为Message Queue,消息队列，保存消息并将它们转发给消费者。

## RabbitMQ整体架构

![](E:\笔记\image\e25102aafa5bf6e02f46d0554f70c020.png)

## RabbitMQ的运行流程

![](E:\笔记\image\9d1972d3931c92e4b629d7bfc06c2608.png)

## RabbitMQ支持消息的模式

### 简单模式 Simple

- 简单队列
- 参考入门案例

### 工作模式 Work

- 类型：无
- 特点：分发机制

### 发布订阅模式fanout

- 类型：fanout
- 特点：Fanout—发布与订阅模式，是一种广播机制，它是没有路由key的模式。
- 操作：创建交换机exchange类型fanout，绑定（Bind）队列（Queue）所有绑定的队列都会收到消息

### 路由模式Direct

- 类型：direct
- 特点：有routing-key的匹配模式
- 操作：创建交换机exchange类型direct，绑定（Bind）队列（Queue）时声明Routing-Key，发送消息时匹配的Routing-Key队列才会接收到消息

### 主题Topic模式

- 类型：topic
- 特点：模糊的routing-key的匹配模式
- 操作：创建交换机exchange类型topic，绑定（Bind）队列（Queue）时声明模糊的Routing-Key，发送消息时可以匹配模糊Routing-Key，匹配的队列才会接收到消息
  模糊Routing-Key：`*`代表一个单词`#`代表0个或多个单词

### 参数模式Header

- 类型：headers
- 特点：参数匹配模式
- header模式与routing不同的地方在于，header模式取消routingkey，使用header中的 key/value（键值对）匹配队列。根据用户的通知设置去通知用户，设置接收Email的用户只接收Email，设置接收sms的用户只接收sms，设置两种通知类型都接收的则两种通知都有效。

# 入门案例

操作：所有操作可用Web界面完成或代码创建

- 创建交换机（exchange）：`exchangeDeclare()`
- 创建队列（queue）：`queueDelete()`
- 绑定交换机与队列关系、key（Bind）：`exchangeBind()`

代码创建：

```java
//9：声明交换机
/**
 * @params1 交换机名称
 * @params2 交换机类型
 * @params3 是否持久化，持久化指交换机会不会随着服务器重启造成丢失
 * */
channel.exchangeDeclare(exchangeName, type, true);
//10：声明队列
/**
 * @params1 队列的名称
 * @params2 是否持久化durable=false 
 * @params3 排他性，是否是独占独立
 * @params4 是否自动删除，随着最后一个消费者消息完毕消息以后是否把队列自动删除
 * @params5 携带的附属参数
 * */
channel.queueDeclare("queue6", true, false, false,null);
//11：绑定队列跟交换机关系
/**
 * @params1 队列名称
 * @params2 交换机名称
 * @params3 RoutingKey
 * */
channel.queueBind("queue6", exchangeName, routeKey);
```

## fanout模式

交换机（exchange）和队列（queue）绑定关系（Bindings）

| To     | Routing Key |
| ------ | ----------- |
| queue1 |             |
| queue2 |             |
| queue3 |             |

**生产者**

```java
            //5：准备发送消息的内容
            String message = "hello";
            //6：准备交换机
            String exchangeName = "fanout-exchange";
            //7：定义路由key
            String routeKey = "";
            //8：指定交换机类型
            String type = "fanout";
            //9：发送消息给交换机
            channel.basicPublish(exchangeName,routeKey,null,message.getBytes());

```

## Direct模式

交换机（exchange）和队列（queue）绑定关系（Bindings）

| To     | Routing Key |
| ------ | ----------- |
| queue1 | email       |
| queue2 | qq          |
| queue3 | wx          |

**生产者**

```java
//5：准备发送消息的内容
String message = "direct message";
//6：准备交换机
String exchangeName = "direct-exchange";
//7：定义路由key
String routeKey = "email";
//8：指定交换机类型
String type = "direct";
//9：发送消息给交换机
channel.basicPublish(exchangeName,routeKey,null,message.getBytes());
```

## Topic模式

交换机（exchange）和队列（queue）绑定关系（Bindings）

| To     | Routing Key  |
| ------ | ------------ |
| queue1 | com.#        |
| queue2 | \*.course.\* |
| queue3 | \#.order.#   |
| queue4 | \#.user.*    |

- `*`代表一个单词`#`代表0个或多个单词

**生产者**

```java
//5：准备发送消息的内容
String message = "topic message";
//6：准备交换机
String exchangeName = "topic-exchange";
//7：定义路由key
String routeKey = "com.course.order.xxx";
//8：指定交换机类型
String type = "topic";
//9：发送消息给交换机
channel.basicPublish(exchangeName,routeKey,null,message.getBytes());
```

## Header模式

交换机（exchange）和队列（queue）绑定关系（Bindings）

| To     | Routing key | Arguments         |
| ------ | ----------- | ----------------- |
| queue1 |             | inform_type:email |
| queue2 |             | inform_type:sms   |

```java
Map<String, Object> headers_email = new Hashtable<String, Object>();
headers_email.put("inform_type", "email");
Map<String, Object> headers_sms = new Hashtable<String, Object>();
headers_sms.put("inform_type", "sms");
channel.queueBind(queueName,exchangeName,"",headers_email);
channel.queueBind(queueName,exchangeName,"",headers_sms);
```

**生产者**

```java
String message = "email inform to user"+i;
Map<String,Object> headers =  new Hashtable<String, Object>();
headers.put("inform_type", "email");//匹配email通知消费者绑定的header
//headers.put("inform_type", "sms");//匹配sms通知消费者绑定的header
AMQP.BasicProperties.Builder properties = new AMQP.BasicProperties.Builder();
properties.headers(headers);
//Email通知
channel.basicPublish(exchangeName, "", properties.build(), message.getBytes());
```



## Work模式

生产者发布20条消息队列

```java
//3：从连接工厂中获取连接Connection
connection = connectionFactory.newConnection("生产者");
//4：通过连接获取通道Channel
channel = connection.createChannel();
//5：准备发送消息的内容
for (int i = 0; i < 20; i++) {
    String message = "message:" + i;
    //6：发送消息给交换机
    channel.basicPublish("","queue1",null,message.getBytes());
}
```

### 轮询模式（Round-Robin）

- 类型：无
- 特点：该模式接收消息是当有多个消费者接入时，消息的分配模式是一个消费者分配一条，直至消息消费完成；



消费者Work1、Work2：autoAck=true，自动应答

自动应答：一旦rabbitmq将消息分发给了消费者，就会将消息从内存中删除。这种情况下，如果正在执行的消费者被“杀死”或“崩溃”，就会丢失正在处理的消息

```java
//传参第二个为autoAck
channel.basicConsume("queue1", true, new DeliverCallback() {
    @Override
    public void handle(String consumerTag, Delivery message) throws IOException {
        System.out.println("Work1收到消息是" + new String(message.getBody(), "UTF-8"));
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}, new CancelCallback() {
    @Override
    public void handle(String consumerTag) throws IOException {
        System.out.println("接收消息失败！");
    }
});
```

消息队列轮流给每个消费者，Work1：1、3、5、7、9.....Work2：0、2、4、6、8......

### 公平分发（Fair Dispatch）

- 类型：无
- 特点：由于消息接收者处理消息的能力不同，存在处理快慢的问题，我们就需要能者多劳，处理快的多处理，处理慢的少处理；

消费者Work1、Work2：basicQos(1)在消费者确认应答前只接收1个Message，自动应答autoAck=false

手动应答：消费者接受并处理完一个消息后，会发送应答给rabbitmq，rabbitmq收到应答后，才会将该条消息从内存中删除。如果一个消费者在处理消息的过程中“崩溃”，rabbitmq没有收到应答，那么”崩溃“前正在处理的这条消息会重新被分发到别的消费者。

basicQos(prefetchCount,global)：

- prefetchCount：会告诉RabbitMQ不要同时给一个消费者推送多于N个消息，即一旦有N个消息还没有ack，则该consumer将block掉，直到有消息ack
- global：true\false 是否将上面设置应用于channel，简单点说，就是上面限制是channel级别的还是consumer级别

```java
finalChannel.basicQos(1);
finalChannel.basicConsume("queue1", false, new DeliverCallback() {
    @Override
    public void handle(String consumerTag, Delivery message) throws IOException {
        try {
            System.out.println("Work2收到消息是" + new String(message.getBody(), "UTF-8"));
            Thread.sleep(200);	//模拟处理快慢
            finalChannel.basicAck(message.getEnvelope().getDeliveryTag(),false);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}, new CancelCallback() {
    @Override
    public void handle(String consumerTag) throws IOException {
        System.out.println("接收消息失败！");
    }
});
```

消息队列按处理能力去分配，一条消息处理完Ack后立马给下一条消息。处理速度分配的消息就多。

# SpringBoot案例

## fanout模式

配置类：声明交换机、队列、绑定关系

```java
@Configuration
public class RabbitMqConfig {
    // 1：声明注册fanout模式的交换机
    @Bean
    public FanoutExchange fanoutExchange() {
        return new FanoutExchange("fanout_order_exchange", true, false);
    }
    //2：声明队列
    @Bean
    public Queue smsQueue() {
        return new Queue("sms.fanout.queue", true);
    }
    @Bean
    public Queue wxQueue() {
        return new Queue("wx.fanout.queue", true);
    }
    @Bean
    public Queue emailQueue() {
        return new Queue("email.fanout.queue", true);
    }
    //3：完成绑定关系
    @Bean
    public Binding smsBinding() {
        return BindingBuilder.bind(smsQueue()).to(fanoutExchange());
    }
    @Bean
    public Binding wxBinding() {
        return BindingBuilder.bind(wxQueue()).to(fanoutExchange());
    }
    @Bean
    public Binding emailBinding() {
        return BindingBuilder.bind(emailQueue()).to(fanoutExchange());
    }
}
```

模拟生产者，发送订单

```java
public void makeOrder (String userId, String productId, int num){
    //1：根据商品ID查询库存是否充足
    //2：保存订单
    String orderId = UUID.randomUUID().toString();
    //3：通过MQ来完成消息的分发
    /**
     * @params1 交换机
     * @params2 路由Key/queue队列名称
     * @params3 消息内容
     * */
    String exchangeName = "fanout_order_exchange";
    String routingKey = "";
    rabbitTemplate.convertAndSend(exchangeName, routingKey, orderId);
}
```

消费者FanoutEmailConsumer、FanoutSMSConsumer、FanoutWxConsumer

```java
@RabbitListener(queues = {"email.fanout.queue"})
@Service
public class FanoutEmailConsumer {

    @RabbitHandler
    public void reviceMessage(String message) {
        System.out.println("email.fanout.queue---收到消息：" + message);
    }
}
```

## direct模式

配置类

```java
@Configuration
public class DirectRabbitMqConfig {
    // 1：声明注册direct模式的交换机
    @Bean
    public DirectExchange directExchange() {
        return new DirectExchange("direct_order_exchange", true, false);
    }
    //2：声明队列
    @Bean
    public Queue smsDirectQueue() {
        return new Queue("sms.direct.queue", true);
    }
    @Bean
    public Queue wxDirectQueue() {
        return new Queue("wx.direct.queue", true);
    }
    @Bean
    public Queue emailDirectQueue() {
        return new Queue("email.direct.queue", true);
    }
    //3：完成绑定关系
    @Bean
    public Binding smsDirectBinding() {
        return BindingBuilder.bind(smsDirectQueue()).to(directExchange()).with("sms");
    }
    @Bean
    public Binding wxDirectBinding() {
        return BindingBuilder.bind(wxDirectQueue()).to(directExchange()).with("wx");
    }
    @Bean
    public Binding emailDirectBinding() {
        return BindingBuilder.bind(emailDirectQueue()).to(directExchange()).with("email");
    }
}
```

## topic模式

在消费者类上通过注解方式声明交换机、队列、绑定关系

```java
@RabbitListener(bindings = @QueueBinding(
        value = @Queue(value = "email.topic.queue", durable = "true", autoDelete = "false"),
        exchange = @Exchange(value = "topic_order_exchange", type = ExchangeTypes.TOPIC),
        key = "#.email.#"
))
@Service
public class TopicEmailConsumer {

    @RabbitHandler
    public void reviceMessage(String message) {
        System.out.println("email.fanout.queue---收到消息：" + message);
    }
}
```

## 事务使用

1. channel.txSelect()声明启动事务模式；
2. channel.txCommint()提交事务；
3. channel.txRollback()回滚事务；

# RabbitMQ—高级

## TTL过期时间

声明队列时设置队列TTL过期时间

```java
@Configuration
public class TTLRabbitMqConfig {
    // 1：声明注册fanout模式的交换机
    @Bean
    public FanoutExchange ttlfanoutExchange() {
        return new FanoutExchange("ttl_fanout_order_exchange", true, false);
    }
    @Bean
    public Queue ttlQueue() {
        //设置过期时间TTL
        Map<String, Object> args = new HashMap<>();
        args.put("x-message-ttl", 5000);
        return new Queue("ttl.fanout.queue", true, false, false, args);
    }
    @Bean
    public Binding ttlBinding() {
        return BindingBuilder.bind(ttlQueue()).to(ttlfanoutExchange());
    }
}
```

发送消息时设置消息TTL过期时间

```java
public void makeOrderTTL (String userId, String productId, int num){
        //1：根据商品ID查询库存是否充足
        //2：保存订单
        String orderId = UUID.randomUUID().toString();
        //3：通过MQ来完成消息的分发
        /**
         * @params1 交换机
         * @params2 路由Key/queue队列名称
         * @params3 消息内容
         * */
        String exchangeName = "fanout_order_exchange";
        String routingKey = "";
        //设置消息过期时间
        MessagePostProcessor messagePostProcessor = new MessagePostProcessor() {
            @Override
            public Message postProcessMessage(Message message) throws AmqpException {
                message.getMessageProperties().setExpiration("5000");
                message.getMessageProperties().setContentEncoding("UTF-8");
                return message;
            }
        };


        rabbitTemplate.convertAndSend(exchangeName, routingKey, orderId, messagePostProcessor);
    }
}
```

## 死信队列

> 概述

DLX，全称为Dead-Letter-Exchange , 可以称之为死信交换机，也有人称之为死信邮箱。当消息在一个队列中变成死信(dead message)之后，它能被重新发送到另一个交换机中，这个交换机就是DLX ，绑定DLX的队列就称之为死信队列。
消息变成死信，可能是由于以下的原因：

- 消息被拒绝
- 消息过期
- 队列达到最大长度

DLX也是一个正常的交换机，和一般的交换机没有区别，它能在任何的队列上被指定，实际上就是设置某一个队列的属性。当这个队列中存在死信时，Rabbitmq就会自动地将这个消息重新发布到设置的DLX上去，进而被路由到另一个队列，即死信队列。
要想使用死信队列，只需要在定义队列的时候设置队列参数 `x-dead-letter-exchange` 指定交换机即可。

![img](https://img-blog.csdnimg.cn/img_convert/3f9a11d5ee5f8d74f49e82869408bb78.png)

声明死信队列

```java
@Configuration
public class DeadRabbitMqConfig {
    // 1：声明注册fanout模式的交换机
    @Bean
    public DirectExchange deadExchange() {
        return new DirectExchange("dead_direct_exchange", true, false);
    }
    @Bean
    public Queue deadQueue() {
        return new Queue("dead.direct.queue", true);
    }
    @Bean
    public Binding deadBinding() {
        return BindingBuilder.bind(deadQueue()).to(deadExchange()).with("dead");
    }
}
```

设置队列的死信队列关系

```java
@Bean
public Queue ttlQueue() {
    //设置过期时间TTL
    Map<String, Object> args = new HashMap<>();
    args.put("x-message-ttl", 5000);
    //设置死信队列关系
    args.put("x-dead-letter-exchange","dead_direct_exchange");
    args.put("x-dead-letter-routing-key","dead");
    return new Queue("ttl.fanout.queue", true, false, false, args);
}
```

## RabbitMQ集群

单机多实例搭建

> 假设两个RabbitMQ结点，Rabbit-1作为主节点，Rabbit-2作为从节点

启动第一个节点

```shell
RABBITMQ_NODE_PORT=5672 RABBITMQ_NODENAME=rabbit-1 rabbitmq-server start &
```

结束命令

```shell
rabbitmqctl -n rabbit-1 stop
```

启动第二个节点，Web管理插件端口占用，所以要指定其Web插件占用端口:RABBITMQ_SERVER_START_ARGS="-rabbitmq_management listener [{port,15673}]"

```
RABBITMQ_NODE_PORT=5673 RABBITMQ_NODENAME=rabbit-2 RABBITMQ_SERVER_START_ARGS="-rabbitmq_management listener [{port,15673}]" rabbitmq-server start &
```

验证服务

```
ps aux|grep rabbitmq
```

rabbit-1操作为主节点

```
#停止应用
> rabbitmqctl -n rabbit-1 stop_app
#目的是清除节点上的历史数据（如果不清除，无法将节点加入到集群）
> rabbitmqctl -n rabbit-1 reset
#启动应用
> rabbitmqctl -n rabbit-1 start_app
```

rabbit-2操作为从节点

```
# 停止应用
> rabbitmqctl -n rabbit-2 stop_app
# 目的是清除节点上的历史数据（如果不清除，无法将节点加入到集群）
> rabbitmqctl -n rabbit-2 reset
# 将rabbit2节点加入到rabbit1（主节点）集群当中【Server-node服务器的主机名】
> rabbitmqctl -n rabbit-2 join_cluster rabbit-1@'Server-node'
# 启动应用
> rabbitmqctl -n rabbit-2 start_app
```

验证集群状态

```
rabbitmqctl cluster_status -n rabbit-1
```

多机部署集群需保证`/var/lib/rabbitmq/.erlang.cookie`多机一致

## 分布式事务

