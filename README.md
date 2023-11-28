# Docker Develop Environment 

Docker 开发环境集成

## 组件

```
* PHP 5.6/7.4
* MySQL 5.7
* Redis 4/5/6
* RabbitMQ 3.9
```

## 快速开始

推荐的目录结构（自行创建数据目录、日志目录）：

```
docker
├── data (数据目录、自行创建)
│   ├── mysql57
│   ├── redis4
│   └── www  
└── image (git clone https://github.com/smallmenu/docker-develop.git)
    ├── app
    │   ├── alpine
    │   │   ├── php56
    │   │   │   └── Dockerfile
    │   │   └── php74
    │   │       └── Dockerfile
    │   └── debian
    │       └── sources.list
    ├── etc (配置目录)
    │   ├── mysql57
    │   │   └── my.cnf
    │   ├── nginx
    │   │   ├── conf.d
    │   │   └── nginx.conf
    │   ├── php56
    │   │   ├── php-fpm.conf
    │   │   ├── php-fpm.d
    │   │   │   └── www.conf
    │   │   └── php.ini
    │   ├── php74
    │   │   ├── php-fpm.conf
    │   │   ├── php-fpm.d
    │   │   │   └── www.conf
    │   │   └── php.ini
    │   └── redis4
    │       └── redis.conf
    └── var （日志目录，自行创建）
        └── log
            ├── nginx
            ├── php56
            ├── php74
            └── redis4
```

## Windows/Mac 开发环境

Windows 安装 Docker 需要 `Windows 10 64-bit: Pro, Enterprise, or Education (Build 16299 or later).`

参见：https://docs.docker.com/docker-for-windows/install/

切换为 Linux 容器模式，自行进行 Docker 初始化配置，包括共享目录， docker 镜像源，网络，主机资源等

### docker0

Windows/Mac 的 Docker 桌面中没有 docker0

### 容器连接主机上的服务

虽然在容器中可以直接 ping 通宿主，但是容器不允许直接与宿主进行服务通讯，因为默认是 bridge 网络。

可以通过 `host.docker.internal` 或 `gateway.docker.internal` 来实现通讯。

Nginx 转发 PHP-FPM，需要使用 host.docker.internal 这个地址，转发到宿主机上，同时 PHP 的网站目录和 Nginx 要保持一致。

```
location ~ \.php$
{
    fastcgi_pass  host.docker.internal:9000;
    fastcgi_index index.php;
    include fastcgi.conf;
}
```

同理，你的 PHP 代码中也需要使用 host.docker.internal 作为主机名 host 来连接其他容器服务。


## 使用方法

主要挂载配置、日志、数据目录。

### 启动 Nginx 1.19

挂载了主配置文件，额外配置目录，日志目录，网站目录。

```
docker run -d --name nginx --privileged -p 80:80 \
-v /data/docker/etc/nginx/conf.d:/etc/nginx/conf.d \
-v /data/docker/etc/nginx/nginx.conf:/etc/nginx/nginx.conf \
-v /data/docker/var/log/nginx:/var/log/nginx \
-v /data/www:/usr/share/nginx/html \
nginx:1.19-alpine
```

### 启动 Redis4

配置外部配置文件 redis.conf，以及数据目录。通过 redis-server 命令指定配置文件启动

```
docker run -d --name redis4 --privileged -p 6379:6379 \
-v /data/docker/etc/redis4/redis.conf:/usr/local/etc/redis/redis.conf \
-v /data/docker/var/log/redis4:/var/log/redis \
-v /data/redis4:/data \
redis:4-alpine redis-server /usr/local/etc/redis/redis.conf
```

### 启动 MySQL 5.7

配置外部配置文件，以及数据目录

```
docker run -d --name mysql57 --privileged -p 3306:3306 \
-v /data/docker/etc/mysql57:/etc/mysql/conf.d \
-v /data/docker/var/log/mysql57:/var/log/mysql \
-v /data/mysql57:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=123123 \
-e TZ='Asia/Shanghai' \
-d mysql:5.7
```

### 手动构建并启动 PHP 7.4 FPM

挂载了 php.ini、php-fpm.conf、日志目录、网站目录。

注意：网站目录需要和 Nginx 的 root 路径保持一致。

```
cd app/alpine/php74
docker build -t local/php:7.4-fpm-alpine .

docker run -d --name php74 --privileged -p 9000:9000 \
-v /data/docker/etc/php74/php.ini:/usr/local/etc/php.ini \
-v /data/docker/etc/php74/php-fpm.conf:/usr/local/etc/php-fpm.conf \
-v /data/docker/etc/php74/php-fpm.d/www.conf:/usr/local/etc/php-fpm.d/www.conf \
-v /data/docker/var/log/php74:/var/log/php \
-v /data/www:/usr/share/nginx/html local/php:7.4-fpm-alpine
```

### 手动构建并启动 PHP 7.4 FPM

挂载了 php.ini、php-fpm.conf、日志目录、网站目录。

注意：网站目录需要和 Nginx 的 root 路径保持一致。

```
cd app/alpine/php74
docker build -t local/php:7.4-fpm-alpine .

docker run -d --name php74 --privileged -p 9000:9000 \
-v /data/docker/etc/php74/php.ini:/usr/local/etc/php.ini \
-v /data/docker/etc/php74/php-fpm.conf:/usr/local/etc/php-fpm.conf \
-v /data/docker/etc/php74/php-fpm.d/www.conf:/usr/local/etc/php-fpm.d/www.conf \
-v /data/docker/var/log/php74:/var/log/php \
-v /data/www:/usr/share/nginx/html local/php:7.4-fpm-alpine
```

### 手动构建并启动 PHP 5.6 FPM

挂载了 php.ini、php-fpm.conf、日志目录、网站目录。

注意：网站目录需要和 Nginx 的 root 路径保持一致。

```
cd app/alpine/php56
docker build -t local/php:5.6-fpm-alpine .

docker run -d --name php56 --privileged -p 9000:9000 \
-v /data/docker/etc/php56/php.ini:/usr/local/etc/php.ini \
-v /data/docker/etc/php56/php-fpm.conf:/usr/local/etc/php-fpm.conf \
-v /data/docker/etc/php56/php-fpm.d/www.conf:/usr/local/etc/php-fpm.d/www.conf \
-v /data/docker/var/log/php56:/var/log/php \
-v /data/www:/usr/share/nginx/html local/php:5.6-fpm-alpine
```

## 启动 RabbitMQ 3.9

```
docker run -d --name rabbitmq39 --privileged -p 9001:9000 \
-e RABBITMQ_DEFAULT_USER=user -e RABBITMQ_DEFAULT_PASS=password \
-v /data/docker/etc/php56/php.ini:/usr/local/etc/php.ini \
rabbitmq:3.9-management-alpine
```