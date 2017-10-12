### prepare

1. install docker 
2. install git

### use dev

1. git clone git@github.com:saqing/dev.git
2. cd dev && docker-compose up

### Then 

1. Open  browser 
2. Visit  localhost:9999
3. You will see phpinfo 

### 目录介绍

1. conf   配置目录，用于配制 nginx http 服务，nginx proxy 服务，redis，php

2. data   数据，用于保存 mysql , redis 数据

3. logs   日志，用于记录 nginx http 服务, nginx proxy 服务，mysql 产生的日志

4. www  项目目录

   ​



### 例子：运行一个单页应用（含php http api）

1. cd dev

2. cd www  && git clone  git@github.com:saqing/demo4dev.git

3. cd conf && cd nginx-proxy

4. touch  demo4dev.conf , edit it

   1. 关键项目编辑
      1. server_name
      2. access_log
      3. error_log
      4. proxy_pass

   ```shell
   #demo4dev.conf
   server {
       listen  80;
       server_name demo4dev.local;

       gzip on;
       gzip_min_length 1k;
       gzip_comp_level 2;
       gzip_types text/plain application/javascript application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png;
       gzip_vary on;
       gzip_disable "MSIE [1-6]\.";

       access_log  /wwwlogs/demo4dev_access.log main;
       error_log   /wwwlogs/demo4dev_error.log;

       location /api {
           proxy_pass          http://nginx-http:8002;
           proxy_redirect      off;
           proxy_set_header    X-Real-IP       $remote_addr;
           proxy_set_header    Host            $http_host;
           proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
       }
        location / {
            proxy_pass          http://nginx-http:8003;
            proxy_redirect      off;
            proxy_set_header    X-Real-IP       $remote_addr;
            proxy_set_header    Host            $http_host;
            proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
         }
   }
   ```

5. cd conf && cd nginx-http

6. touch demo4dev-api.conf  , edit it

   1. 关键编辑项
      1. listen
      2. set $root_path
      3. access_log
      4. error_log

   ```
   #demo4dev
   server {
           listen  8002;
           server_name localhost;
           set $root_path '/www/demo4dev/api/public';
           root $root_path;
           index  index.php index.html index.htm;
           access_log  /wwwlogs/demo4dev-api_access.log main;
           error_log   /wwwlogs/demo4dev-api_error.log;


           try_files $uri $uri/ @rewrite;

           location @rewrite {
                       rewrite ^/(.*)$ /index.php?_url=/$1;
                   }

           location ~ \.php {
               fastcgi_pass php-fpm:9000;
               fastcgi_index /index.php;

               fastcgi_split_path_info       ^(.+\.php)(/.+)$;
               fastcgi_param PATH_INFO       $fastcgi_path_info;
               fastcgi_param PATH_TRANSLATED $document_root$fastcgi_path_info;
               fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
               include                       fastcgi_params;
           }

   }
   ```

7. touch demo4dev-web.conf  , edit it

   1. 关键编辑项
      1. listen
      2. set $root_path
      3. access_log
      4. error_log

   ```
   server {
           listen  8003;
           server_name localhost;
           set $root_path '/www/demo4dev/web/dist';
           root $root_path;
           index  index.html index.htm;
           access_log  /wwwlogs/demo4dev-web_access.log main;
           error_log   /wwwlogs/demo4devd-web_error.log;


           try_files $uri $uri/ @rewrite;

           location @rewrite {
                       rewrite ^/(.*)$ /index.html?_url=/$1;
                   }

   }
   ```

   ​

8. cd  && sudo vi /etc/hosts

   1. add host record  e.g.  127.0.0.1  demo4dev.local

9. cd   && cd dev && docker-compose restart

10. Everything is OK,try visit   http://demo4dev.local:9999   and   http://demo4dev.local:9999/api  

