# elasticsearch安装

    wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.4.0.zip
    unzip elasticsearch-5.4.0.zip
    
    useradd elk
    
    chown -R elk:elk elasticsearch-5.4.0
    sudo su elk
    ./bin/elasticsearch -Xmx1g -Xms1g

# 配置参数解决外网无法访问
    解决办法：vim config/elasticsearch.yml

    增加：network.host: 0.0.0.0

    重启问题解决
    
    切换root账户 vim /etc/sysctl.conf
    增加一行  vm.max_map_count=655360
    接着执行 sysctl -p
    切回ES账户重新启动问题解决

# 解除linux系统的最大进程数和最大文件打开数限制
    vi /etc/security/limits.conf
    确保：
    root soft nofile 65536
    root hard nofile 65536
    * soft nofile 65536
    * hard nofile 65536
    
    修改 /etc/security/limits.d/90-nproc.conf 
    *          soft    nproc     unlimited
    root       soft    nproc     unlimited
