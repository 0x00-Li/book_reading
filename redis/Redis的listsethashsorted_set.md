# Redis的list/set/hash/sorted_set



## List

### lpush rpush

`lpush k1 a b c d e f` 表示从左边开始放，真实存放为 fedcba

`rpush k2 a b c d e f`表示忘链表后追加 真实存放为 abcdef



### lpop rpop



> 栈：同向命令
>
> 队列：反向命令



### lrange

> 存在正负索引

范围取出链表总的值

### lindex

取指定索引的值

### lset

向指定位置设置值，针对下标操作，可以作为数组

### lrem

移除指定元素

正负数表示，倒数两个，正数表示前两个



### linsert

在指定位置插入元素



### llen

长度



### blpop

> 可以实现阻塞，单播队列



### ltrim

对两端进行删除

## hash

> 保存的值为键值对

场景：每个用户有三四个维度，怎么用redis存取



### hset



### hget





### hmget



### hkeys



### hvals



### hgetall



### hincrbyfloat

### hincrby

### 总结

1. 可以进行数值计算

## set

> list 是可重复的， 是有序的（保存序）

1. 无序
2. 去重
3. 随机性

### sadd



### scard



### sdiff

差集

有方向性

获取前面的集合数，第二个作为参考

### srem

删除元素



### sinter

直接打印交集

### sinterstore

交集结果会存在一个目标key



### sunion

并集



### smembers

获取所有元素，影响网卡性能

### srandmember

随机事件



正数的时候，尽量满足要求，去重结果集（不能超过已有集合）

负数的时候，取出带有重复的结果集，一定满足你的数量

场景：抽奖、家庭争斗



### spop

随机取出一个



## sorted_set

> 排序依据

1. 元素
2. 分值
3. 索引

正反向索引

## zadd



物理内存的顺序，左小右大的分值顺序；并且顺序并不会随着命令发生变化



### zrange



### zrangebyscore 

按照分值区间读取



### zrevrange

按照分数从高到低取出



### zscore



### zrank

取出排名

### zincrby





## 集合操作

### zunionstore

最后设置聚合参数



### sorted_set 排序怎么实现的

skip_list 实现的



