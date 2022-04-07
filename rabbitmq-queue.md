## RabbitMQ Queue

rabbitmq 中的queue是FIFO


### 属性：


Name

Durable (broker重启，队列还会存在)

Exclusive (队列只被一个connection使用，connection关闭队列就会被删除)

Auto-delete (至少有一个consumer的队列，最后一个consumer unsubscribes 时会被删除)

Arguments (可选的; used by plugins and broker-specific features such as message TTL, queue length limit, etc)


### 声明

queue在被使用前需要被声明，声明的queue如果不存在，queue就会被创建。如果声明的queue已经存在且声明的属性和存在的queue的属性相同，则声明不产生影响。反之，channel-level exception with code 406 (PRECONDITION_FAILED) 就会出现。


### Optional arguments

大多数参数都可以在queue声明后更改，但也有例外。比如，queue type (x-queue-type) and max number of queue priorities (x-max-priority) 必须在声明时设置，之后不可更改。


### 消息顺序

FIFO 在 priority and sharded queues 不能被保证。


competing consumers, consumer priorities, message redeliveries 也会影响消息的顺序。任何的redeliveries都适用：automatic after channel closure and negative consumer acknowledgements.

redelivered设置为false，可以认为消息顺序可以保证，repeated deliveries (redelivered设置为true)情况下，消息的顺序会受到consumer acknowledgements and redeliveries的影响，因此不能保证顺序。

多个consumer时，如果所有的consumer有相同的priority，会采取round-robin策略，只有consumer没有超过prefetch value才会被考虑。


### Temporary Queues

三种queue会被自动删除：

Exclusive queues 

TTLs 

Auto-delete queues


### Exclusive Queues


Exclusive Queues 只能在声明它的connection被使用。声明它的connection关闭时，队列会被删除。


### Time-to-Live and Length Limit

queue 和 message 有ttl，用于数据过期，queue最多可以用多少资源(RAM, disk space) 


### Priorities


queue可以有0或多个priorities

publisher通过priority属性确定message priority

如果priority 队列被声明了，1到10比较推荐使用，更多的priorities会消耗更多的资源



