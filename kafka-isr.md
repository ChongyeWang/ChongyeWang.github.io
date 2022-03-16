## Kafka ISR

ISR：In-Sync Replica

ack：

acks=0：producer不等待broker返回的ack，producer不会知道消息是否丢失也不会重新发送消息。

acks=1：leader收到消息后，producer会收到ack。消息丢失只会发生在leader在ack了消息后立刻失效，但还没来的及同步给follower。

acks=all：producer得到ack在所有in-sync replicas收到消息的时候。leader会等待所有isr确认收到消息。

AR：所有副本

ISR：与leader保持一定同步的副本。

ISR是AR的子集。

如果一个follower 失效，超过默认时间，会被移除ISR，如果由于网络问题或服务器资源限制，如果落后leader超过默认时间，也会被移除ISR。
