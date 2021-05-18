# Redis的 NIO

常识：

数据是存在磁盘中：

1. 寻址 ms
2. 带宽 G/M

内存：

1. 寻址ns
2. 带宽很大

秒>ms>微秒>纳秒 磁盘比内存寻址慢了10w倍



I/O buffer :成本问题

磁盘与磁道，扇区，一扇区512byte 带来成本变大：索引

4K操作系统，无论你读多少，都是4k从磁盘拿

## 数据库存储

> 基于常识的分析，数据，从磁盘读取会慢很多

1. 关系型数据库，建表：必须给出schema，类型，字段宽度，倾向于行级存储（方便改动，不移动其他数据）

2. 数据库：表很大，性能下降？

   ​	如果有索引，增删改慢，查询速度呢？1个少量查询依然很快；并发打的时候会受硬盘带宽影响速度

   > SAP HANA 内存级别的数据库
   >
   > 数据在磁盘和内存的体积不一样



基于磁盘和内存之前，这种方案：缓存！！memcached,redis

2个基础设施：冯诺依曼体系的硬件；以太网，tcp/IP的网络



## Redis 基础

数据类型：String(字符类型，数值类型，bitmap), Hashs, Lists ,Sets, Sorted Sets

> memcached value 没有类型的概念；存储复杂的数据接口的时候，value使用json存储
>
> Value 有类型的价值：
>
> 1. 获取kv中v的某一条数据：memcache 返回value的所有数据到client，占用io
> 2. 类型不是很重要，Redis的Server中对没中类型有自己的方法

> 计算向数据移动：减少io，提高效率

## 安装

> centos 6.x redis 5.x

### 源码编译

`make `

`make install [prefix=target dictory]`

编译失败,清理产生的文件：`make distclean`

在`utils/insetall_server.sh`下进行，处理环境变量，服务安装：

​	会自动的配置启动的程序，以及相关配置（端口，日志，持久化目录，自动启动配置等）





## 原理



单进程，单线程，单实例：并发很高；如何变的很快？

使用epoll 模型

> epoll 模型
>
> BIO：每个线程进行blocking 读取 网络文件描述符
>
> NIO（同步非阻塞）：轮训文件描述符，处理可用的网络IO。C10K问题，带进来的成本问题
>
>  
>
> 多路复用的NIO：kernel 发展，提供select系统调用，返回ready状态的网络IO，然后再处理具体的网络IO；但是每次需要传递n多个网络IO（文件描述符），用户态和内核态进行多次交互。
>
>  
>
> 共享空间（mmap）：存放红黑树，链表；维护网络IO状态



> zero copy:系统调用sendfile(out,in)

​	

> nginx 多少课cpu启动多少个worker进程，每个进程使用多路复用NIO