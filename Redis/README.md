# Redis

参考：[黑马程序员Redis](https://www.bilibili.com/video/BV1cr4y1671t)  

推荐指数：★★★★



## 并发问题

+ 缓存穿透
  + 查询为空也存到数据库中，设置短暂的ttl
  + [布隆过滤器](https://blog.csdn.net/belongtocode/article/details/103376219)
+ 缓存雪崩
  + 缓存时间+随机时间
  + 集群模式
+ 缓存击穿(热点key问题)
  + 互斥锁
  + 逻辑锁



## 锁

1. 悲观锁，简单的乐观锁，数据库的间隙锁(store > 0)
2. redis的setnx+值必须存储该jvm+线程标识，防止误删
3. 为了解决设置锁和获取锁中间时产生上下文切换，导致非原子性，添加了[lua](https://www.zhihu.com/question/346626417)脚本（可以批量操作redis，因为redis是单线程，所以是原子性的）
4. redission
   + 可重入，synchronized使用的count属性记录，Lock锁中使用的state属性记录，redission使用的hash记录，重入一次就加一，释放一次就-1 ，直到减少成0 时，表示当前这把锁没有被人持有
   + 重试，使用的是发布订阅，类似wait和notify，删除锁的时候notify，等待的时候订阅
   + 锁超时，WatchDog机制，默认锁失效时间30s，过了1/3就会递归调用刷新失效时间
   + 主从一致性，为了解决主从复制中主机宕机，可能引发锁失效。MutiLock连锁，就是向多个独立的redis节点，必须所有节点都获取重入锁，才算获取锁成功



## 秒杀

1. 新增秒杀优惠卷的同时，将优惠券信息保存到Redis中
2. 基于Lua脚本，判断秒杀库存、一人一单，决定用户是否抢购成功
3. 如果抢购成功，将优惠券id和用户id封装后存入消息队列
