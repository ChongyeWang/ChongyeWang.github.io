## Redis 设计与实现(Note)

SDS：比起C字符串，SDS具有以下优点:

1)常数复杂度获取字符串长度。

2)杜绝缓冲区溢出。 

3)减少修改字符串长度时所需的内存重分配次数。

4)二进制安全。

5)兼容部分C字符串函数。

<br />
链表：

链表被广泛用于实现Redis的各种功能，比如列表键、发布与订阅、慢查询、监视器等。 

每个链表节点由一个listNode结构来表示，每个节点都有一个指向前置节点和后置节点的指针，所以Redis的链表实现是双端链表。 

每个链表使用一个list结构来表示，这个结构带有表头节点指针、表尾节点指针，以及链表长度等信息。 

因为链表表头节点的前置节点和表尾节点的后置节点都指向NULL，所以Redis的链表实现是无环链表。

通过为链表设置不同的类型特定函数，Redis的链表可以用于保存 各种不同类型的值。  

<br />
字典：

字典被广泛用于实现Redis的各种功能，其中包括数据库和哈希键。

Redis中的字典使用哈希表作为底层实现，每个字典带有两个哈希表，一个平时使用，另一个仅在进行rehash时使用。

当字典被用作数据库的底层实现，或者哈希键的底层实现时，Redis使用MurmurHash2算法来计算键的哈希值。

哈希表使用链地址法来解决键冲突，被分配到同一个索引上的多个键值对会连接成一个单向链表。

在对哈希表进行扩展或者收缩操作时，程序需要将现有哈希表包 含的所有键值对rehash到新哈希表里面，并且这个rehash过程并不是一 次性地完成的，而是渐进式地完成的。

<br />

跳跃表：

跳跃表是有序集合的底层实现之一。

Redis的跳跃表实现由zskiplist和zskiplistNode两个结构组成，其中 zskiplist用于保存跳跃表信息(比如表头节点、表尾节点、长度)，而 zskiplistNode则用于表示跳跃表节点。

每个跳跃表节点的层高都是1至32之间的随机数。 

在同一个跳跃表中，多个节点可以包含相同的分值，但每个节点的成员对象必须是唯一的。

跳跃表中的节点按照分值大小进行排序，当分值相同时，节点按 照成员对象的大小进行排序。

<br />
整数集合：

整数集合是集合键的底层实现之一。

整数集合的底层实现为数组，这个数组以有序、无重复的方式保 存集合元素，在有需要时，程序会根据新添加元素的类型，改变这个数 组的类型。

升级操作为整数集合带来了操作上的灵活性，并且尽可能地节约 了内存。

整数集合只支持升级操作，不支持降级操作。

<br />

压缩列表：

压缩列表是一种为节约内存而开发的顺序型数据结构。

压缩列表被用作列表键和哈希键的底层实现之一。 

压缩列表可以包含多个节点，每个节点可以保存一个字节数组或者整数值。

添加新节点到压缩列表，或者从压缩列表中删除节点，可能会引 发连锁更新操作，但这种操作出现的几率并不高。

<br />
<br />



Redis服务器的所有数据库都保存在redisServer.db数组中，而数据库的数量则由redisServer.dbnum属性保存。 

客户端通过修改目标数据库指针，让它指向redisServer.db数组中的不同元素来切换不同的数据库。 

数据库主要由dict和expires两个字典构成，其中dict字典负责保存键值对，而expires字典则负责保存键的过期时间。 

因为数据库由字典构成，所以对数据库的操作都是建立在字典操作之上的。

数据库的键总是一个字符串对象，而值则可以是任意一种Redis对 象类型，包括字符串对象、哈希表对象、集合对象、列表对象和有序集 合对象，分别对应字符串键、哈希表键、集合键、列表键和有序集合 键。

expires字典的键指向数据库中的某个键，而值则记录了数据库键 的过期时间，过期时间是一个以毫秒为单位的UNIX时间戳。

Redis使用惰性删除和定期删除两种策略来删除过期的键:惰性删 除策略只在碰到过期键时才进行删除操作，定期删除策略则每隔一段时 间主动查找并删除过期键。

执行SAVE命令或者BGSAVE命令所产生的新RDB文件不会包含已 经过期的键。

执行BGREWRITEAOF命令所产生的重写AOF文件不会包含已经过 期的键。

当一个过期键被删除之后，服务器会追加一条DEL命令到现有 AOF文件的末尾，显式地删除过期键。

当主服务器删除一个过期键之后，它会向所有从服务器发送一条 DEL命令，显式地删除过期键。

从服务器即使发现过期键也不会自作主张地删除它，而是等待主 节点发来DEL命令，这种统一、中心化的过期键删除策略可以保证主从 服务器数据的一致性。

当Redis命令对数据库进行修改之后，服务器会根据配置向客户端 发送数据库通知。

<br />
<br />

RDB文件用于保存和还原Redis服务器所有数据库中的所有键值对数据。

SAVE命令由服务器进程直接执行保存操作，所以该命令会阻塞服务器。

BGSAVE令由子进程执行保存操作，所以该命令不会阻塞服务器。

服务器状态中会保存所有用save选项设置的保存条件，当任意一个 保存条件被满足时，服务器会自动执行BGSAVE命令。

RDB文件是一个经过压缩的二进制文件，由多个部分组成。 

对于不同类型的键值对，RDB文件会使用不同的方式来保存它们。

<br />
<br />

AOF文件通过保存所有修改数据库的写命令请求来记录服务器的数据库状态。

AOF文件中的所有命令都以Redis命令请求协议的格式保存。

命令请求会先保存到AOF缓冲区里面，之后再定期写入并同步到 AOF文件。

appendfsync选项的不同值对AOF持久化功能的安全性以及Redis服 务器的性能有很大的影响。

服务器只要载入并重新执行保存在AOF文件中的命令，就可以还 原数据库本来的状态。

AOF重写可以产生一个新的AOF文件，这个新的AOF文件和原有 的AOF文件所保存的数据库状态一样，但体积更小。

AOF重写是一个有歧义的名字，该功能是通过读取数据库中的键 值对来实现的，程序无须对现有AOF文件进行任何读入、分析或者写入 操作。

在执行BGREWRITEAOF命令时，Redis服务器会维护一个AOF重 写缓冲区，该缓冲区会在子进程创建新AOF文件期间，记录服务器执行 的所有写命令。当子进程完成创建新AOF文件的工作之后，服务器会将 重写缓冲区中的所有内容追加到新AOF文件的末尾，使得新旧两个AOF文件所保存的数据库状态一致。最后，服务器用新的AOF文件替换旧的 AOF文件，以此来完成AOF文件重写操作。



