### 1 什么是消息队列

1. 消息队列首先是一个队列，既然是队列，就有FIFO的性质。消息队列的最大好处就是实现异步。
2. 使用消息队列的好处。
   - 解耦，生产者和消费者的逻辑可以通过消息队列隔开，进行解耦。
   - 削峰填谷，在生产者速度很快的时候，消息队列可以为消费者缓冲，在生产者很慢的时候，消费者仍然能以较大的速度处理队列中的任务。
   - 提高效率，对于生产者来说，不需要等待消费者，可以提高效率。对于消费者来说亦是如此。
   - 广播，生产者只要生产一次，可以让多个消费者收到消息。
3. 使用消息队列带来的弊端
   - 提高了系统的复杂性。需要防止消息丢失等问题。
   - 暂时的不一致性。生产者生产完之后，消费者不一定能马上处理这个消息，生产者也无法确定消费者是否收到了这个消息。所以生产者和消费者之间产生了不一致。
4. 使用消息队列的场景
   - 能够容许暂时的不一致性。
   - 生产者不需要从消费者处得到反馈，即可以异步执行。
   - 确实能受享受到解耦、削峰填谷、提速、广播带来的好处。



### 2 什么是RabbitMQ

1. AMQP

   AMQP是一个应用层协议，它为消息队列地实现提供了一个标准，而RabbitMQ实现了这个标准。

2. RabbitMQ有哪些组件

   - Producer：生产者
   - Exchange：交换器，生产者把消息交给Exchange的时候，一般会给一个RoutingKey来指定路由规则。
   - binding：绑定，这里指Queue和Exchange之间的绑定关系，绑定时一般会指定一个BindingKey。
   - Queue：用于存储消息
   - Connection和Channel：Connection代表一条TCP连接，而Channel建立在Connection中，多个Channel可以复用同一条连接。Channel是我们与RabbitMQ打交道的最重要的一个接口，我们大部分的业务操作是在Channel这个接口中完成的，包括定义Queue、定义Exchange、绑定Queue与Exchange、发布消息等。
   - Consumer：消费者



### 3 RabbitMQ的工作模式

1. Producer在发送消息时可以指定Exchange和routingKey，Exchanger指定发送到哪个exchange，routingKey指定发送到绑定到这个Exchange的哪个队列。
2. Exchange可以指定三种转发路由模式：
   - direct：直接将routingKey和bindingKey比较。
   - fanout：直接转发到所有绑定的队列，无需routingkey和bindingKey。
   - topic：bindingKey可以使用通配符模式与routingKey匹配。
3. Queue需要与Exchange绑定，绑定的时候可以指定bindingKey，用于和routingKey匹配。
4. Consumer：一个消费者可以接收来自多个消息队列的消息。一个队列的消息也可以被多个消费者接收。

使用上述性质可以组成不同模式的消息队列。



### 4 RabbitMQ消息的可靠性

1. 持久化消息

   持久化消息在到达队列时会被持久化到磁盘，如果消息队列服务器发生重启，可以恢复持久化消息。

2. 消息回执

   如果消费者在拿到消息后，还没来得及处理就发生了宕机，那么这个消息就会丢失。这种情况可以要求消费者在处理完消息后给RabbitMQ发送一个回执，RabbitMQ只有收到这个回执之后才会删除这条消息。

3. 生产消息确认机制

   生产者如何确定消息有没有到达Exchange。

   - 可以通古AMQP提供的事务机制实现

     ```java
     try {
     	channel.txSelect(); // 声明事务
     	// 发送消息
     	channel.basicPublish("", _queueName, MessageProperties.PERSISTENT_TEXT_PLAIN, message.getBytes("UTF-8"));
     	channel.txCommit(); // 提交事务
     } catch (Exception e) {
     	channel.txRollback();
     } finally {
     	channel.close();
     	conn.close();
     }
     ```

   - 可以通通过Confirm发送方确认模式实现

     ```java
     // 开启发送方确认模式
     channel.confirmSelect();
     String message = String.format("时间 => %s", new Date().getTime());
     channel.basicPublish("", config.QueueName, null, message.getBytes("UTF-8"));
     if (channel.waitForConfirms()) {
     	System.out.println("消息发送成功" );
     }
     ```

     