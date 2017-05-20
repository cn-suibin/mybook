
    #user  nobody;
    worker_processes  auto;

    error_log  logs/error.log;
    #error_log  logs/error.log  notice;
    #error_log  logs/error.log  info;

    #pid        logs/nginx.pid;


    events {
        worker_connections  204800;
        multi_accept on;
        use epoll;
    }


    http {
        include       mime.types;
        default_type  application/octet-stream;

        #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
        #                  '$status $body_bytes_sent "$http_referer" '
        #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #    access_log  logs/access.log  main;

        sendfile        on;
        #tcp_nopush     on;

        #keepalive_timeout  0;
        keepalive_timeout  65;

        #gzip  on;
        gzip on;
            gzip_disable "msie6";
            gzip_proxied any;
            gzip_min_length 1000;
            gzip_buffers  4 8k;
            gzip_comp_level 6;
            gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript;
            open_file_cache max=204800 inactive=20s;
            open_file_cache_valid 30s;
            open_file_cache_min_uses 2;
            open_file_cache_errors on;
        upstream test {
            # fake server otherwise ngx_http_upstream will report error when startup
            server 127.0.0.1:11111;

            # all backend server will pull from consul when startup and will delete fake server
    #        upsync 127.0.0.1:8500/v1/kv/upstreams/test upsync_timeout=6m upsync_interval=500ms upsync_type=consul strong_dependency=off;
    #        upsync_dump_path /usr/local/nginx/conf/servers/servers_test.conf;
        }

        upstream bar {
            server 127.0.0.1:8090 weight=1 fail_timeout=10 max_fails=3;
        }
        server {
            listen 80;
            server_name gw.360lzy.com;
            index index.php index.html index.htm;
            root /home/webroot/demo/public;
            location / {
                try_files $uri $uri/ /index.php?$query_string;
            }
            location ~ ^(.+\.php)(.*)$
            {
                    fastcgi_split_path_info ^(.+\.php)(.*)$;
                    include fastcgi.conf;
                    fastcgi_pass unix:/dev/shm/php-cgi.sock;
                    #fastcgi_pass 127.0.0.1:9000;
                    fastcgi_index index.php;
                    fastcgi_param PATH_INFO $fastcgi_path_info;
                    fastcgi_ignore_client_abort   on;
            }


        }

        server {
            listen       80;
            server_name  www.360lzy.com 360lzy.com;
            index index.php index.html index.htm;
            root   /home/webroot;
            #charset koi8-r;

            #access_log  logs/host.access.log  main;

            location / {
                    try_files $uri $uri/ /index.php?_url=$uri&$args;
            }


            location = /proxy_test {
                proxy_pass http://test;
            }

            location = /bar {
                proxy_pass http://bar;
            }
            location /upstream_list {
                upstream_show;
            }
            #error_page  404              /404.html;

            # redirect server error pages to the static page /50x.html
            #
            error_page   500 502 503 504  /50x.html;
            location = /50x.html {
                root   html;
            }

            # proxy the PHP scripts to Apache listening on 127.0.0.1:80
            #
            #location ~ \.php$ {
            #    proxy_pass   http://127.0.0.1;
            #}

            # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
            #
            location ~ ^(.+\.php)(.*)$
            {
                    fastcgi_split_path_info ^(.+\.php)(.*)$;
                    include fastcgi.conf;
                    #fastcgi_pass 127.0.0.1:9000;
                    fastcgi_pass unix:/dev/shm/php-cgi.sock;
                    fastcgi_index index.php;
                    fastcgi_param PATH_INFO $fastcgi_path_info;
                    fastcgi_ignore_client_abort   on;
            }
            # deny access to .htaccess files, if Apache's document root
            # concurs with nginx's one
            #
            #location ~ /\.ht {
            #    deny  all;
            #}
        }


        # another virtual host using mix of IP-, name-, and port-based configuration
        #
        #server {
        #    listen       8000;
        #    listen       somename:8080;
        #    server_name  somename  alias  another.alias;

        #    location / {
        #        root   html;
        #        index  index.html index.htm;
        #    }
        #}


        # HTTPS server
        #
        #server {
        #    listen       443 ssl;
        #    server_name  localhost;

        #    ssl_certificate      cert.pem;
        #    ssl_certificate_key  cert.key;

        #    ssl_session_cache    shared:SSL:1m;
        #    ssl_session_timeout  5m;

        #    ssl_ciphers  HIGH:!aNULL:!MD5;
        #    ssl_prefer_server_ciphers  on;

        #    location / {
        #        root   html;
        #        index  index.html index.htm;
        #    }
        #}

    }
