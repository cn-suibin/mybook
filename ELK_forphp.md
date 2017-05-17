# elasticsearch安装

    wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.4.0.zip
    unzip elasticsearch-5.4.0.zip
    useradd elk
    chown -R elk:elk elasticsearch-5.4.0
    ./bin/elasticsearch -Xmx1g -Xms1g
