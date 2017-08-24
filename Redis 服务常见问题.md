# Redis 服务常见问题



## Redis被bgsave和bgrewriteaof阻塞的解决方法

Redis 是一个性能非常高效的内存 Key-Value 存储服务, 同时它还具有两个非常重要的特性: 1. 持久化; 2. Value 数据结构. 这两个特性让它在不少场景轻松击败了 Memcached 和 Casandra 等.

Redis 是一个性能非常高效的内存 Key-Value 存储服务, 同时它还具有两个非常重要的特性: 1. 持久化; 2. Value 数据结构. 这两个特性让它在不少场景轻松击败了 Memcached 和 Casandra 等.

Redis 的持久化在两种方式: Snapshotting(快照) 和 Append-only file(aof). 在一个采用了 aof 模式的 Redis 服务器上, 当执行 bgrewriteaof 对 aof 进行归并优化时, 出现了 Redis 被阻塞的问题, 此时, Redis 无法提供任何读取和写入操作.

按字面理解, bgrewriteaof 是在后台进行操作, 不应该影响 Redis 的正常服务. 原理也确实是这样的, Redis 首先 fork 一个子进程, 并在该子进程里进行归并和写持久化存储设备(如硬盘)的. 按照正常逻辑, 在一台多核的机器上, 即使子进程占满 CPU 和硬盘, 也不应该导致 Redis 服务阻塞啊!

其实, 问题就出在硬盘上.

Redis 服务设置了 appendfsync everysec, 主进程每秒钟便会调用 fsync(), 要求内核将数据”确实”写到存储硬件里. 但由于子进程同时也在写硬盘, 从而导致主进程 fsync()/write() 操作被阻塞, 最终导致 Redis 主进程阻塞了.

解决方法便是设置 no-appendfsync-on-rewrite yes, 在子进程处理和写硬盘时, 主进程不调用 fsync() 操作. 注意, 即使进程不调用 fsync(), 系统内核也会根据自己的算法在适当的时机将数据写到硬盘(Linux 默认最长不超过 30 秒).