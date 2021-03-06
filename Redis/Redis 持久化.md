# Redis 持久化
**RDB持久化**
RDB持久化是将当前进程中的数据生成快照保存到硬盘(因此也称作快照持久化)，保存的文件后缀是rdb；当Redis重新启动时，可以读取快照文件恢复数据。 那么RDB持久化的过程，相当于在执行bgsave命令

**AOF持久化**
RDB持久化是将进程数据写入文件，而AOF持久化(即Append Only File持久化)，则是将Redis执行的每次写命令记录到单独的日志文件中。

## Redis持久化方案

1. RDB(默认)
* RDB是redis默认的持久化方式
* RDB是采用快照的方式来进行数据持久化的,当符合快照的条件时Redis会自动对内存中的数据进行快照,然后持久化到硬盘中
* 触发条件
    * 符合自定义配置的快照规则
    * 执行save或者bgsave命令
        * eg : save 60 10000 ：表示1分钟内至少100个键被更改则进行快照。
    * 执行flushall命令
    * 执行主从复制操作
    * 在redis.conf中设置快照规则
        * 打开redis.config文件,202行,这是RDB的默认配置,满足下面三个之一就会被触发
        > save 900 1
        > save 300 10
        > save 60 10000
        * 在247行,配置快照生成地址
        > dir ./
        * 在237行可以配置生成快照的名称,默认是dump.rdb
        > dbfilename dump.rdb
* 每次redis启动时都会去读取dump.rdb快照文件,将数据加载到内存中
原理 : Redis使用fork函数复制一份当前进程的副本(创建一个子进程)父进程继续接收处理客户端的请求,子进程负责将内存中的数据存储到硬盘中,当子进程写入完所有数据之后会用新的rdb文件替换旧的rdb文件
* 细节 : 快照的时候并不会修改原有的rdb文件,而是用新生成的替换旧的,所以rdb文件是一定存在的,rdb文件是经过压缩的二进制文件,所以占用内存会很少,并且方便读取
* 优点 : 因为是复制除了一个子进程来实现数据存储,所以父进程还是可以继续响应客户端的请求,所以客户端基基本不会受到影响,而且rdb是压缩后的二进制文件,进行数据恢复的速度会比较快,所以rbd非常适合用来做数据备份
* 缺点 : 必须要满足rdb的条件才会执行数据备份,所以有可能因为Redis的突然宕机导致部分数据丢失,所以设置快照条件时必须足够严谨

作者：高哲
链接：https://zhuanlan.zhihu.com/p/58265935