## 本地开发环境

数据库

* MySQL  5.7
* MySQL  8.0
* MongoDB 3.6
* RethinkDB 2.3
* Redis 3.2
* Redis 4.0

其他

* Nginx  1.14
* WAMP Router



应用环境

- 几个命令

  - docker-compose up -d   启动一组容器
  - docker-compose stop    关闭一组容器
  - docker-sync  start           打开文件同步
  - docker-sync  stop            关闭文件同步
  - docker restart  nginx      重启 nginx
  - docker exec -it app /bin/bash  进入应用容器
- 两个目录
  - code  存放代码
  - nginx-conf 存放 nginx 配置
- 连接数据库
 - MySQL
   - 5.7   root:root@MySQL5.7  PORT 3306
   - 8.0   root:root@MySQL8.0  PORT 3306
 - Redis
   - 3.2   HOST:Redis3.2  PORT 6379   NO Password
   - 4.0   HOST:Redis4.0  PORT 6379   NO Password
 - MongoDB
   - 3.6  root:root@MongoDB3.6  PORT 27017 
 - RethinkDB
   - 2.3  HOST:RethinkDB2.3  PORT 27015   NO Password
- Websocket 
  - WAMP 协议 
    - 从浏览器中连接   ws://127.0.0.1:8080/ws
    - 从容器中连接       ws://crossbar:8080/ws




## 如何开始

准备工作

 ```
1. 安装 docker edge Version 18.05.0 
参考 https://store.docker.com/editions/community/docker-ce-desktop-mac

2. 安装 docker-sync  
参考 https://github.com/EugenMayer/docker-sync/wiki
 ```



环境搭建

```
1. git clone git@github.com:saqing/dev.git
2. cd dev && docker-compose up -d
3. docker-sync start
```



开始一个项目

```
1.创建一个项目    cd code && git clone git@github.com:saqing/dev-example.git
2.配置 nginx     cd nginx-conf && vi dev-example.conf
3.重启 nginx     docker restart nginx
4.更新 host      在 /etc/hosts 添加 127.0.0.1  dev.example.local
5.在浏览器访问 dev.example.local ，你可以看到 phpinfo 信息
```



nginx 配置参考

```
# dev-example
server {
    listen  80;
    server_name dev.example.local;
    set $root_path '/www/code/dev-example/public';
    root $root_path;
    index  index.php index.html index.htm;

    try_files $uri $uri/ @rewrite;

    location @rewrite {
        rewrite ^/(.*)$ /index.php?_url=/$1;
    }

    location ~ \.php {
        fastcgi_pass app:9000;
        fastcgi_index /index.php;
        fastcgi_split_path_info       ^(.+\.php)(/.+)$;
        fastcgi_param PATH_INFO       $fastcgi_path_info;
        fastcgi_param PATH_TRANSLATED $document_root$fastcgi_path_info;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include                       fastcgi_params;
    }

    location ~* ^/(css|img|js|flv|swf|download)/(.+)$ {
        root $root_path;
    }

    location ~ /\.ht {
        deny all;
    }
}
```







  ​


