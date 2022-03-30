## Elasticsearch 读写数据过程


### 写数据

- 客户端向node发送请求，这个node被称为协调节点

- 协调节点对document进行路由，将请求发送给有primary shard 的node

- primary shard 处理请求，成功后，将数据同步到副本分片

- 所有primary node和replica node报告成功后，协调节点返回响应结果给客户端



### 读数据

- 客户端发送请求到node（协调节点）

- 协调节点根据document的_id进行哈希路由，将请求转发到对应的node，通过轮询，在primary shard和replica shard中随机选择一个，来达到负载均衡

- 接受请求的node将document返回给协调节点

- 协调节点将document返回给客户端
