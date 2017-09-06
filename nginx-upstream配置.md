   upstream backend{
        server 192.168.248.243:8091;
 #       server 192.168.248.243:8091 weight=2;
   }

    server{
        listen 8090;
        server_name 192.168.248.243;
        location /{
             #设置主机头和客户端真实地址，以便服务器获取客户端真实IP
             proxy_set_header Host $host;
             proxy_set_header X-Real-IP $remote_addr;
             proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
             #禁用缓存
             proxy_buffering off;
             proxy_pass http://backend;
        }
    }

    server {
		listen       8091;
		server_name  192.168.248.243;
		root /home/renwu/;
		index index.php;

		location / {
		try_files $uri $uri/ /index.php?$query_string;

		}

        location ~ ^(.+\.php)(.*)$
        {
          fastcgi_index  index.php;
          fastcgi_param   SCRIPT_FILENAME $document_root$fastcgi_script_name;
        ####
          fastcgi_split_path_info ^(.+\.php)(.*)$;
          fastcgi_param   PATH_INFO               $fastcgi_path_info;
          fastcgi_param   PATH_TRANSLATED $document_root$fastcgi_path_info;
        ####
          fastcgi_pass   127.0.0.1:9000;
          include        fastcgi_params;

        }
    }
