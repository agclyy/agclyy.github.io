---
title: redis 学习
date: 2019-05-03 11:57:08
categories: redis
tags: redis
---

### 1、下载安装

​	可直接官网下载 ： https://redis.io/download

​	目前最新稳定版本是：redis-5.0.4

```shell
[root@localhost test]# wget http://download.redis.io/releases/redis-5.0.4.tar.gz
[root@localhost test]# tar xzf redis-5.0.4.tar.gz
[root@localhost test]# cd redis-5.0.4/
[root@localhost test]# make
```

### 2、启动测试

```shell
[root@localhost redis-5.0.4]# ./src/redis-server
```

​	再启动一个终端，进行客户端的连接测试

```sh
[root@localhost src]# ./redis-cli 
127.0.0.1:6379> set test aa
OK
127.0.0.1:6379> get test
"aa"
127.0.0.1:6379> 
```

### 3、常见问题

- 异常一

  ```shell
  make[2]: cc: Command not found
  ```

  异常原因：没有安装gcc

  解决方案：`yum install gcc-c++`

- 异常二

  ```shell
  zmalloc.h:51:31: error:jemalloc/jemalloc.h: No such file or directory
  ```

  异常原因：一些编译依赖或原来编译遗留出现的问题

  解决方案：`make distclean`。清理一下，然后再`make`.

- 异常三

  在make成功以后，需要make test。在make test出现异常。couldn't execute "tclsh8.5": no such file or directory

  异常原因：没有安装tcl

  解决方案：`yum install -y tcl`。

**redis安装依赖包包括：gcc，jemalloc，tcl，cloog-ppl，cpp，mpfr，ppl**

### 4、哨兵模式配置

​	copy三份redis.conf的配置，主节点配置文件

```shell
bind 0.0.0.0  #使非本机可以访问
protected-mode no #保护模式关闭，3.2版本之前的注释该行
port 6379   #访问端口
daemonize yes  #redis启动后台运行
pidfile "/var/run/redis_6379.pid"  #后台启动时将pid写入文件
databases 16  #数据库个数默认16
dir "/apps/svr/test/redis-5.0.4"  #数据文件保持地址

# slaveof <masterip> <masterport>
#设置主节点的ip,端口
# masterauth <master-password>
masterauth "testpassword" #主备通信时告诉备需要密码
requirepass "testpassword" #设置登陆需要的密码
```

​	从节点配置文件

```shell
bind 0.0.0.0  #使非本机可以访问
protected-mode no #保护模式关闭，3.2版本之前的注释该行
port 6380   #访问端口
daemonize yes  #redis启动后台运行
pidfile "/var/run/redis_6380.pid"  #后台启动时将pid写入文件
databases 16  #数据库个数默认16
dir "/apps/svr/test/redis-5.0.4"  #数据文件保持地址

# slaveof <masterip> <masterport>
slaveof 127.0.0.1 6379
#slave-read-only yes
#设置主节点的ip,端口
# masterauth <master-password>
masterauth "testpassword" #主备通信时告诉备需要密码
requirepass "testpassword" #设置登陆需要的密码
```

配置好以后，启动redis

```shell
[root@localhost redis-5.0.4]# nohup ./src/redis-server redis_6381.conf
[root@localhost redis-5.0.4]# ps aux|grep redis
root     37286  0.0  0.0  60696  2000 ?        Ss   14:01   0:00 sudo -u redis redis-server /etc/redis.conf
systemd+ 37438  0.2  0.0  43712  5724 ?        Sl   14:01   0:00 redis-server *:6379
root     38153  0.1  0.0 156440  5188 pts/0    Sl   14:02   0:00 ./src/redis-server 127.0.0.1:6379
root     38607  0.1  0.0 156440  8064 ?        Ssl  14:02   0:00 ./src/redis-server 127.0.0.1:6380
root     38781  0.0  0.0 156440  9876 ?        Ssl  14:02   0:00 ./src/redis-server 127.0.0.1:6381
root     38935  0.0  0.0 112704   964 pts/0    S+   14:03   0:00 grep --color=auto redis 
```

查看启动后主节点信息

```shell
[root@localhost redis-5.0.4]# ./src/redis-cli -h 127.0.0.1 -p 6379 
127.0.0.1:6379> auth testpassword
OK
127.0.0.1:6379> get test
(nil)
127.0.0.1:6379> set test aa
OK
127.0.0.1:6379> info
```

info输入后我们可以看到这样的输出

```shell
# Replication
role:master
connected_slaves:2
slave0:ip=127.0.0.1,port=6380,state=online,offset=376,lag=1
slave1:ip=127.0.0.1,port=6381,state=online,offset=376,lag=1
master_replid:63e7bfb237d100f9f68b1342a71b10ce31f5a380
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:376
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:376
```

可以看到role是master 两个slaves 

主、从节点sentinel配置文件

```shell
protected-mode no #3.2版本之前的注释该行
port 26379
#sentinel monitor <master-name> <ip> <redis-port> <quorum>
#主redis地址端口 #设置访问密码
sentinel monitor mymaster 127.0.0.1 6379 2 
sentinel auth-pass mymaster testpassword 
```

启动sentinel，查看

```shell
[root@localhost redis-5.0.4]# ./src/redis-sentinel sentinel_6379.conf &
[root@localhost redis-5.0.4]# ./src/redis-sentinel sentinel_6380.conf &
[root@localhost redis-5.0.4]# ./src/redis-sentinel sentinel_6381.conf &
```

报错No such master with specified name  原来我配置文件的位置弄错了，下面两句的配置需要放在

sentinel.conf的大致位置如下，之前我直接配置在了文件最末端

`sentinel monitor mymaster 127.0.0.1 6379 2` 
`sentinel auth-pass mymaster testpassword` 

```shell
# sentinel down-after-milliseconds <master-name> <milliseconds>
#
# Number of milliseconds the master (or any attached replica or sentinel) should
# be unreachable (as in, not acceptable reply to PING, continuously, for the
# specified period) in order to consider it in S_DOWN state (Subjectively
# Down).
#
# Default is 30 seconds.
sentinel monitor mymaster 127.0.0.1 6379 2

# sentinel parallel-syncs <master-name> <numreplicas>
#
# How many replicas we can reconfigure to point to the new replica simultaneously
# during the failover. Use a low number if you use the replicas to serve query
# to avoid that all the replicas will be unreachable at about the same
# time while performing the synchronization with the master.
sentinel auth-pass mymaster testpassword

# sentinel failover-timeout <master-name> <milliseconds>
```

访问sentinel

```shell
[root@localhost redis-5.0.4]# ./src/redis-cli -p 26379
127.0.0.1:26379> info
# Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=mymaster,status=ok,address=127.0.0.1:6379,slaves=2,sentinels=3
```

可以测试kill主节点，你会发现其中的一个slave变成了master,配置文件的内容也会跟着发生变化

再启动曾经的master，会自动成为新的master的slave ，配置文件也会更新

配置主从 新旧版本配置不一样 具体查看redis官网
        新版本为：replicaof 127.0.0.1 6379
        旧版本为: slaveof 127.0.0.1 6379

之前配置的是slaveof ，自动修改后成为了 replicaof 127.0.0.1 6380

### 5、参考链接&鸣谢

1. https://blog.csdn.net/wuliusir/article/details/51598029
2. https://blog.csdn.net/gzh0222/article/details/8482525
3. https://www.cnblogs.com/PatrickLiu/p/8444546.html
4. https://www.cnblogs.com/jaycekon/p/6237562.html
5. https://blog.csdn.net/weixin_42711549/article/details/83061052