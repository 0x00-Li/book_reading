# Redis的消息订阅、pipeline 事务 modules

redis进阶使用

redis作为数据库、和缓存的区别



缓存的常见问题



redis的持久化

## 管道 pipeline

批量发送命令，减少网络成本



## 消息订阅   发布订阅

先订阅后，再发布才能收到；



## 事务



multi 开启事务：随后发送的命令都会放入队列，哪个队列先进来exec先执行哪个事务   

exec 执行事务

watch 类似乐观锁机制，提前监听，指定的key，如果key发生变更，会导致事务不执行





## modules



redis-server --loadmodule /绝对路径/redisbloom





### redisbloom

场景：

防止缓存穿透，提前设置缓存中bloom



counting bloom





布谷鸟过滤器





## redis 作为数据库/缓存的区别

### 缓存

1. 缓存数据，没那么重要
2. 缓存不是全量数据
3. 缓存应该随着访问变化
4. 热数据

**redis作为缓存的时候：**

redis里面的数据，怎么随着业务变化，只保留热数据；因为内存大小有限

key的有效期：

1. 受业务逻辑推动控制
2. 内存受限，随着业务的运转，应该淘汰掉冷数据



内存淘汰策略：

> lru 最近最少使用
>
> lfu 最少使用次数

- noeviction 直接反馈错误
- allkeys-lru 对所有的key执行lru
- volatile-lru  对进行了过期的时间进行lru
- allkeys-random
- volatile-random
- volatile-ttl



### 有效期

1. 不会因为访问而延长时间
2. 发生写key，会剔除过期时间的限制
3. 可以设置在什么时候，过期expireat，且redis不能延长
4. 定时

**过期判断原理：**被动和主动方式

被动方式：访问的时候判定

主动方式：周期轮训判定（增量）



真实：

1. 随机测试20个key
2. 删除所有已经过期

3. aa

目的，牺牲内存，保住性能为王！！！

