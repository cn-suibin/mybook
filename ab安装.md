
```
1.独立安装

ab运行需要依赖apr-util包，安装命令为： 

 

yum install apr-util
yum install httpd-tools

简单使用说明
1. 最基本的关心两个选项 -c -n
例： ./ab -c 100 -n 10000 http://127.0.0.1/index.php

-c 100 即：每次并发100个
-n 10000 即： 共发送10000个请求

```
