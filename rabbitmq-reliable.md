## RabbitMQ 可靠传输

### 生产者丢数据解决：

1. 开启事务机制（效率低）

2. confirm发送方确认机制

```
Channel channel = connection.createChannel();
channel.confirmSelect();
```


事务机制是同步的，confirm机制是异步的。


### RabbitMQ丢数据解决：


步骤1：将队列的持久化标识durable设置为true（队列持久化）


步骤2：发送消息的时候设置 deliveryMode = 2 （消息持久化）


### 消费者丢数据解决：

使用手动确认消息

```
boolean autoAck = false;
channel.basicConsume(queueName, autoAck, "a-consumer-tag",
     new DefaultConsumer(channel) {
         @Override
         public void handleDelivery(String consumerTag,
                                    Envelope envelope,
                                    AMQP.BasicProperties properties,
                                    byte[] body)
             throws IOException
         {
             long deliveryTag = envelope.getDeliveryTag();
             // positively acknowledge a single delivery, the message will
             // be discarded
             channel.basicAck(deliveryTag, false);
         }
     });
```
