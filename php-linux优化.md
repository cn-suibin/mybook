=========================nginx 优化==============================
```


worker_processes  auto;
worker_rlimit_nofile 100000;

error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  204800;
    multi_accept on;
    use epoll;
}
```





=======================PHP 优化==============================

```
error_log = /app/logs/php-fpm.log  #指定pid路径
log_level = error                        #开启日志，log级别为error
events.mechanism = epoll         #使用epoll模式
listen.owner = www                    #使用php的用户
listen.group = www
pm.max_children = 1024            #php子进程数量
pm.start_servers = 14                #php初始启动子进程数量
pm.min_spare_servers = 5        #php最小空闲进程数量
pm.max_spare_servers = 20        #php最大空闲进程数量
pm.process_idle_timeout = 15s;    #进程超时时间
pm.max_requests = 2048            #每个子进程退出之前可以进行的请求数
slowlog = /app/logs/$pool.log.slow    #开启慢查询日志(执行程序时间长了可以查看到)
rlimit_files = 32768                        #开启文件描述符数量
request_slowlog_timeout = 10        #慢查询的超时时间，超时10秒记录
```




==========================linux 优化========================

/etc/sysctl.conf 

```
ulimit -n


vi /etc/security/limits.conf

* soft nproc 65535
* hard nproc 65535
* soft nofile 65535
* hard nofile 65535
```
