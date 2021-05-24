# zookeeper 原理知识，paxos 、zab、角色功能



## 回顾知识点











## 重点

paxos

zab

watch

api：不怕写zkclient

callback -> reactive 响应式编程(更加充分的压榨os，HW资源，性能，快速)



> 分布式协调：扩展，可靠性，时序性

## 扩展性

### 框架架构

角色：leader follower observer

### 读写分离

只有follower 才能选举

Observer 放大查询能力

### zoo.cfg

基础配置



## 可靠性

> 攘其外必先安其内

快速恢复leader

数据，可靠可用，一致性

​	1. 攘其外，一致性？最终一致性【过程中，节点是否对外提供服务！！！暂停对外提供】





## Paxos

定义，故事示例



https://www.douban.com/note/208430424/



## ZAB

> 原子广播协议：是paxos的缩减版

作用在可用状态，有Leader可用

原子：成功，失败。没有中间状态（队列+2PC）

广播：分布式多节点，全部知道！！

队列：FIFO ，顺序性



往客户端写数据的时候：

​		只要过半确认可写入，就执行写入命令；最终响应客户端





## 选主

> 集群第一次启动；重启集群；leader挂了

成为leader的条件：

1. 经验最丰富的（数据最全的，zxid最大的）
2. myid 最大的

！！！过半通过的数据才是真数据，你见到的可用zxid



zk选举过程：

1. 3888 造成亮亮通信
2. 只要任何人投票，都会触发那个准leader发起自己投票
3. 推选制：先比较zxid，如果zxid相同，再比较myid



## Watch

> 观察

watch的注册只发生在get和exists操作上



服务发现的实现：

1. 心跳：自己实现
2. watch 基于zk实现

> 方向性、时效性

事件类型：

1. create
2. delete
3. change
4. children

> 事件触发后，会产生回调callback



watch 分类：

1. new zk的时候，传入外退出，是session级别的，跟path，node没关系





会话漂移，会保留

## API







