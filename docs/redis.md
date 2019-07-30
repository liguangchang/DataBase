# redis

[TOC]



### 数据类型

| **数据类型** | **赋值**              | **取值**                       | **删除**              |                       |
| ------------ | --------------------- | ------------------------------ | --------------------- | --------------------- |
| String       | Set   Mset            | Get   Mget                     | del                   | Incr/decr             |
| Hash         | Hset   hmset          | Hget   hmget                   | hdel                  | hlen key              |
| list         | Lpsuh   rpush         | Lrange list 0 -1               | Lpop list   Rpop list | llen key              |
| Set          | sadd                  | Smembers set                   | Srem set a            | sismember  key member |
| zset         | Zadd zset 88 zs 76 ls | Zrevrange zset 0 -1 withscores | Zrem zset li          |                       |

```
EXPIRE key seconds			设置key的生存时间（单位：秒）key在多少秒后会自动删除
TTL key 					查看key的生存时间
PERSIST key				清除生存时间
PEXPIRE key milliseconds	生存时间设置单位为：毫秒
keys*
exists key
rename key key_new 重命名
type 返回值类型
```

```
ping  pong 测试连接是否存活
select 0-15 切换数据库
quit 退出连接
echo 打印内容
dbsize 返回当前数据库中key 的数目。
info 获取服务器的信息和统计。
flushdb 删除当前数据库key
flushall 删除所有数据库 key

```

### 持久化

将数据从内存同步到硬盘，都配置会优先架子啊aof

#### RDB(不能保证完整性)

```
RDB方式的持久化是通过快照（snapshotting）完成的，当符合一定条件时Redis会自动将内存中的数据进行快照并持久化到硬盘。
RDB是Redis默认采用的持久化方式，在redis.conf配置文件中默认有此下配置：
save 900 1  #900秒内容如果超过1个key被修改，则发起快照保存
save 300 10 #300秒内容如超过10个key被修改，则发起快照保存
save 60 10000 #表示60 秒内如果超过10000个key被修改，则发起快照保存
在redis.conf中：
	配置dir指定rdb快照文件的位置
	配置dbfilenam指定rdb快照文件的名称
Redis启动后会读取RDB快照文件，将数据从硬盘载入到内存。
```

#### AOF(影响性能)

```
默认情况下Redis没有开启AOF（append only file）方式的持久化，可以通过appendonly参数开启：
appendonly yes
开启AOF持久化后每执行一条会更改Redis中的数据的命令，Redis就会将该命令写入硬盘中的AOF文件。
AOF文件的保存位置和RDB文件的位置相同，都是通过dir参数设置的，默认的文件名是appendonly.aof，可以通过appendfilename参数修改：appendfilename appendonly.aof
appendfsync always     #每次有数据修改发生时都会写入AOF文件。
appendfsync everysec  #每秒钟同步一次，该策略为AOF的缺省策略。
appendfsync no          #从不同步。高效但是数据不会被持久化。

```

#### 主从复制(master 死了 slave只能读 保证完整统一性)

Master写入数据复制到slave，不会阻塞master，在同步数据时，master 可以继续处理client 请求

```
修改从redis服务器上的redis.conf文件，添加slaveof 主redisip  主redis端口
```

##### 首次完整复制过程

```
1、slave 服务启动，slave 会建立和master 的连接，发送sync 命令。
2、master启动一个后台进程将数据库快照保存到RDB文件中
3、master 就发送RDB文件给slave
4、slave 将文件保存到磁盘上，然后加载到内存恢复
5、master把缓存的命令转发给slave
```

##### 部分复制

```
从机连接主机后，会主动发起 PSYNC 命令，
从机会提供 master的runid(机器标识，随机生成的一个串) 和 offset（数据偏移量，
如果offset主从不一致则说明数据不同步），
主机验证 runid 和 offset 是否有效，
runid 相当于主机身份验证码，用来验证从机上一次连接的主机，
如果runid验证未通过则，则进行全同步，
如果验证通过则说明曾经同步过，根据offset同步部分数据。
```

#### 序列化

```
#TODO
```

#### 缓存穿透

```
一般的缓存系统，都是按照key去缓存查询，如果不存在对应的value，就应该去后端系统查找（比如DB）。一些恶意的请求会故意查询不存在的key,请求量很大，就会对后端系统造成很大的压力。这就叫做缓存穿透。
如何避免？
1：对查询结果为空的情况也进行缓存，缓存时间设置短一点，或者该key对应的数据insert了之后清理缓存。
2：对一定不存在的key进行过滤。可以把所有的可能存在的key放到一个大的Bitmap中，查询时通过该bitmap过滤。
```

#### 缓存雪崩

```
当缓存服务器重启或者大量缓存集中在某一个时间段失效，这样在失效的时候，会给后端系统带来很大压力。导致系统崩溃。
如何避免？
1：在缓存失效后，通过加锁或者队列来控制读数据库写缓存的线程数量。比如对某个key只允许一个线程查询数据和写缓存，其他线程等待。
2：做二级缓存，A1为原始缓存，A2为拷贝缓存，A1失效时，可以访问A2，A1缓存失效时间设置为短期，A2设置为长期
3：不同的key，设置不同的过期时间，让缓存失效的时间点尽量均匀。
```

#### redis分布式锁

```
先拿setnx来争抢锁，抢到之后，再用expire给锁加一个过期时间防止锁忘记了释放。
如果在setnx之后执行expire之前进程意外crash或者要重启维护了，那会怎么样？
set指令有非常复杂的参数，这个应该是可以同时把setnx和expire合成一条指令来用的！
```

#### redis集群搭建

##### 集群细节

```
所有节点互相ping 监测是否存活
节点失效要超过半数投票容错决定
集群中一个节点fail 集群fail，映射不完整
集群是将所有的物理节点映射到[0-16383]slot上,cluster 负责维护node<->slot<->value
```

```
Redis 集群中内置了 16384 个哈希槽，
当需要在 Redis 集群中放置一个 key-value 时，
redis 先对 key 使用 crc16 算法算出一个结果，然后把结果对 16384 求余数，
这样每个 key 都会对应一个编号在 0-16383 之间的哈希槽，
redis 会根据节点数量大致均等的将哈希槽映射到不同的节点
```

```
集群因为投票选举容错至少需要三台，保证高可用需要每台配置slave
```

##### 集群搭建

```
1、使用ruby脚本搭建集群。需要ruby的运行环境。
安装ruby
yum install ruby
yum install rubygems

2、安装ruby脚本运行使用的包。
[root@upload ~]# gem install redis-3.0.0.gem 
Successfully installed redis-3.0.0
1 gem installed
Installing ri documentation for redis-3.0.0...
Installing RDoc documentation for redis-3.0.0...
[root@localhost ~]# 

[root@localhost ~]# cd redis-3.0.0/src
[root@localhost src]# ll *.rb
-rwxrwxr-x. 1 root root 48141 Apr  1  2015 redis-trib.rb

```

```
创建实例 修改redis.conf配置文件。配置文件中还需要把cluster-enabled yes前的注释去掉。
```

```
切换到*.rb目录 ./redis-trib.rb create --replicas 1 ip:port ip:port ip:port ip:port ip:port ip:port
```

```
cluster info     打印集群的信息
cluster nodes   列出集群当前已知的所有节点(node)，以及这些节点的相关信息  

```

