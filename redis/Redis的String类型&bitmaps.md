# Redis的String类型&bitmaps

上节的NIO状态下的分析：

一个线程的成本 默认1MB

1. 线程多了调度成CPU浪费
2. 内存成本



Redis顺序性操作：每连接内的命令顺序



## String基础命令



### set



### mset



### mget

### SETRANGE

正向索引

反向索引

### getrange



### STRLEN

### type





### OBJECT

#### object encoding

## 数值类型的操作

> 高并发的场景下的单个值的变化

### incrby





### DECR

### decrby



### incrbyfloat



## 二进制安全

字符流

字节流：redis只读取字节流，不会读取字符流



## bitmap



### setbit offset value



### bitpos



### bitcount

统计1出现的次数

### bitop

位操作



### 使用场景

1. 用户系统，统计用户登录天数，且窗口随机（一年中的每天，对应一位，进行统计）
2. 活跃用户统计（日期作为key，每个用户id对应的位上的标识）

