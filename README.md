# mybook by suibin
#服务器架构相关
Apache BookKeeper
http://blog.csdn.net/plunger2011/article/details/37812843

http://www.4wei.cn/archives/category/linux

1、用好thinkphp框架自带的。

	1.1用好-动态缓存（第一次查询缓存，有则从缓存取，一般用于一个事务中非实时性数据，设计依赖一个事务的生命周期。（比如排行榜，几小时更新一次，可以缓存，设置队列来控制大小））实时性粒度小，组合强
	
	1.2用好-查询缓存（直接缓存select语句的结果集，无需自己设计缓存逻辑）针对数据库压力。多用在目录、菜单、商品列表等。
	
	1.3开启-SQL解析缓存（全局，数据库ORM提速）
	
	1.4用好-静态缓存（通过配置，对制定控制器生成html静态文件，一般是页面做静态化）多用在有页面的情况下。接口方式主要考虑1.1和1.2
	
	1.5用好-原子性、等幂性。如：商品是有数量，避免卖单高并发情况下数量多减，一种情况是用MYSQL innodb 事务对单条商品设置排它锁。一种是并发量特别高时，我们用redis SETNX分布式锁。


2、代码自身优化到极限时候，考虑架构优化

        2.1 服务器情况，php相关服务情况，接口监控。

        2.2 php slow log分析，找出问题原因。80%不是代码的问题，是资源问题。
	
        2.3 RPC异步。由于php-fpm是阻塞式进程，直接RPC调用如果对方系统并发低，进程会等待调用成功或超时。如果一个大量外部接口调用的系统，这个时候php-fpm大量进程处于阻塞，超出服务器处理能力很容易出现502。这里采用异步概念，将PRC远程调用的逻辑用另外一套系统处理，比如：调用接口的逻辑，写在swoole task中，返回数据在swoole的 callback中，这样后面的代码会继续执行或这个控制器不会一直等待。
	
	
3、架构优化极限的时候，考虑架构升级

        3.1更换底层，比如:crontab做任务调度维护很困难，用laravel任务调度来实现。定时任务不要用php-fpm来执行。
   
        3.2 流量管理、自动降级服务、健全高可用。
   
```
解决：
禁止slowlog

vim php-fpm.conf
;request_slowlog_timeout = 10s
;slowlog = /usr/local/log/php-fpm/ckl-slow.log
修改最大执行时间：

vim php.ini
max_execution_time = 60
重启进程：

/etc/init.d/php-fpm reload
等待一段时间，发现一切正常。
查看TCP连接相关：

# ss -s
Total: 287 (kernel 380)
TCP:   597 (estab 122, closed 563, orphaned 0, synrecv 0, timewait 5630/0), ports 577
 
Transport Total     IP        IPv6
*         380       -         -        
RAW       0         0         0        
UDP       1         1         0        
TCP       34        34        0        
INET      35        35        0        
FRAG      0         0         0
同时发现系统TIMEWAIT 较多，所以优化了一些内核相关参数

# sysct -p
bash: sysct: command not found
[root@sapi etc]# sysctl -p
net.ipv4.ip_forward = 0
net.ipv4.conf.default.rp_filter = 1
net.ipv4.conf.default.accept_source_route = 0
net.ipv4.tcp_max_tw_buckets = 6000
net.ipv4.ip_local_port_range = 1024 65000
net.ipv4.tcp_tw_recycle = 1
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_max_orphans = 262144
net.ipv4.tcp_keepalive_time = 1200
net.ipv4.tcp_keepalive_intvl = 30
net.ipv4.tcp_keepalive_probes = 3
net.ipv4.tcp_synack_retries = 1
net.ipv4.tcp_syn_retries = 2
net.ipv4.tcp_max_orphans = 262144
net.ipv4.tcp_max_syn_backlog = 262144
net.ipv4.tcp_timestamps = 0
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216
net.ipv4.tcp_rmem = 4096 87380 4194304
net.ipv4.tcp_wmem = 4096 16384 4194304
net.core.wmem_default = 8388608
net.core.rmem_default = 8388608
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216
net.core.somaxconn = 262144
kernel.sysrq = 0
kernel.core_uses_pid = 1
kernel.msgmnb = 65536
kernel.msgmax = 65536
kernel.shmmax = 68719476736
kernel.shmall = 4294967296
vm.swappiness = 0
fs.file-max = 409600
过一阵再查看：

# ss -s
Total: 281 (kernel 362)
TCP:   520 (estab 22, closed 493, orphaned 0, synrecv 0, timewait 493/0), ports 475
 
Transport Total     IP        IPv6
*         362       -         -        
RAW       0         0         0        
UDP       1         1         0        
TCP       27        27        0        
INET      28        28        0        
FRAG      0         0         0
```
