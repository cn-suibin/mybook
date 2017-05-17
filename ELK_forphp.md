


# 一、elasticsearch安装

    wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.4.0.zip
    unzip elasticsearch-5.4.0.zip
    
    useradd elk
    
    chown -R elk:elk elasticsearch-5.4.0
    sudo su elk
    ./bin/elasticsearch -d
    
    netstat -ant |grep 9200  //查看端口存在说明启动成功
    

#  1.1配置参数解决外网无法访问
    解决办法：vim config/elasticsearch.yml

    增加：network.host: 0.0.0.0

    重启问题解决
    
    切换root账户 vim /etc/sysctl.conf
    增加一行  vm.max_map_count=655360
    接着执行 sysctl -p
    切回ES账户重新启动问题解决

# 1.2解除linux系统的最大进程数和最大文件打开数限制
    vi /etc/security/limits.conf
    确保：
    root soft nofile 65536
    root hard nofile 65536
    * soft nofile 65536
    * hard nofile 65536
    
    修改 /etc/security/limits.d/90-nproc.conf 
    *          soft    nproc     unlimited
    root       soft    nproc     unlimited

# 1.3因为Centos6不支持SecComp，关闭

    解决：
    在elasticsearch.yml中配置bootstrap.system_call_filter为false，注意要在Memory下面:
    bootstrap.memory_lock: false
    bootstrap.system_call_filter: false

# 1.4修改虚拟机内存

    vim config/jvm.options 
    -Xms2g
    -Xmx2g
    
#  1.5安装集群监控器
    
    从https://github.com/mobz/elasticsearch-head下载ZIP包。
     git clone https://github.com/mobz/elasticsearch-head.git  

    cd elasticsearch-head

    npm install

    npm run start &
    
    http://localhost:9100/

# 二、Logstash 安装
    
    wget https://artifacts.elastic.co/downloads/logstash/logstash-5.4.0.tar.gz









参考：http://www.sojson.com/blog/85.html
