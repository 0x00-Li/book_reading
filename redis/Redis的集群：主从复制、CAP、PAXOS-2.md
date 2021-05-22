# Redis的集群：主从复制、CAP、PAXOS-2



## 回顾

akf

xyz扩展

主从复制

单节点：容量问题





redis横向扩展的方式：

1. 逻辑拆分
2. 算法：hash+取模（modula）
3. 逻辑random lpush  ；消费端，直接rpop (相当于消息队列，topic-partition)
4. 一致性哈希算法kemata,（映射算法：hash crc16 crc32 fnv md5），data和node都要参加；规划一个环形的哈希环
   1. 优点：增加节点，的确可以分担其他节点的压力，不会造成全局洗牌
   2. 缺点：新增节点造成一小部分数据不能命中
   3. 问题：击穿，压到mysql
   4. 方案：没去取离我最近的2个屋里节点
   5. 虚拟节点解决，数据倾斜问题



> 粗暴的横向扩容，对server端造成的造成连接成本高；增加代理层，代理层可以无状态的实现modula/random/kemata
>
> 代理方案： vip  lvs keepalived proxy

代理：twemproxy/predixy/cluster





**后三种的模式，不适合做数据库**



预分区进行解决：



## cluster

无主模型

预分配槽位

每个节点存储，集群内槽位和节点的映射关系



>  数据分治问题：很难实现聚合操作、事务



hash tag:解决数据分散问题