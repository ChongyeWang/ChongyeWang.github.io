## Kafka 静态成员（Static Membership）



什么是Rebalance：其实rebalance是对资源和成员间的一种分配。

Rebanlance发生的条件：消费者数量变化，分区数数量变化（partition的数量只能增加不能减少），主题数发生变化。

在整个Rebalance过程中，在分区没有分配完成，consumer是不会消费任何数据的。所以，如果只有一个consumer短暂失效，会导致所有consumer不能消费。

为了避免这种情况，Kafka 2.3 引入了静态成员。

通过group.instance.id这个参数，每个consumer都有了独立的标识，如果一个consumer短暂失效，在达到session.timeout.ms前，broker coordinator是不会通知其他consumer需要rebalance的。

当consumer返回的时候，broker coordinator会返回cached的分配，并不需要rebalance。

当使用静态成员的时候，建议将session.timeout.ms设置到足够大，这样rebalance不会太频繁。








