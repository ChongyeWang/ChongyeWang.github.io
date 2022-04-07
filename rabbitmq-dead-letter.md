## 死信队列

### Overview

当以下三种情况发生时，一个queue中的消息会成为dead-lettered，因此被重新发送到一个exchange：

- messsage被consumer negatively acknowledged （basic.reject or basic.nack with requeue parameter set to false）

- message过期

- message被dropped由于queue超过了长度限制


queue过期不会导致里面的消息成为死信

Dead letter exchanges (DLXs) 是普通的exchange，可以是任何type。

### 定义DLX

- specify a DLX using policy
```
rabbitmqctl set_policy DLX ".*" '{"dead-letter-exchange":"my-dlx"}' --apply-to queues
```
- 为queue设置死信交换机。specify the optional x-dead-letter-exchange argument

```
channel.exchangeDeclare("some.exchange.name", "direct");

Map<String, Object> args = new HashMap<String, Object>();
args.put("x-dead-letter-exchange", "some.exchange.name");
channel.queueDeclare("myqueue", false, false, false, args);
```

### Routing Dead-Lettered Messages

Dead-lettered messages 被路由到 dead letter exchange either:

- 如果设置了routing key，则使用
- 如果没有设置routing key，则是用原来的routing key

e.g. 有routing key foo，死信被发送到dead letter exchange with routing key foo。
如果设置了x-dead-letter-routing-key 为bar，则会被发送到dead letter exchange with routing key bar。


### 安全

Dead-lettered messages 被 re-published 没有打开publisher confirms，因此在集群中使用DLX不保证安全性。

消息在被发送到DLX目标队列中时，会立刻被原来的queue删除。


