# ConcurrentHashMap

## 加锁的方法

putVal 只在具体操作具体的Node的时候对，根Node进行加锁

![ConcurrentHashMap_PutVal](/Users/zero/Documents/book_reading/jdk/concurrenthashmap_putval.png)

Clear 方法也进行了加锁，也是只对具体的node进行了加锁

![clear](/Users/zero/Documents/book_reading/jdk/concurrenthashmap_clear.png)



