=======================================================
```
nginx 优化

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





=====================================================



/etc/sysctl.conf 


==================================================

ulimit -n


vi /etc/security/limits.conf

* soft nproc 65535
* hard nproc 65535
* soft nofile 65535
* hard nofile 65535
`
