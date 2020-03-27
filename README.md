# Docker Windows Develop Environment

## 快速开始

* 安装 Docker Desktop Installer
* 切换为 Linux 容器模式，自行进行 Docker 初始化配置，包括共享目录， docker 镜像源，网络，主机资源等
* clone 项目到 D:/Docker
* 创建数据目录 D:/DockerData

## 注意事项

### docker0

Windows 的 Docker 桌面中没有 docker0，它存在于 Windows 的 Hyper-V 虚拟机中

### 容器连接主机上的服务

虽然在容器中可以直接 ping 通主机，但是现在不允许直接进行服务通讯。

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

## 使用方法

### 启动 Nginx

挂载了主配置文件，额外配置目录，日志目录，网站目录

```
docker run -d --name nginx --privileged -p 80:80 -v D:/Docker/etc/nginx/conf.d:/etc/nginx/conf.d -v D:/Docker/etc/nginx/nginx.conf:/etc/nginx/nginx.conf -v D:/Docker/var/log/nginx:/var/log/nginx -v D:/DockerData/www:/usr/share/nginx/html nginx:alpine
```

### 启动 Redis4

配置外部配置文件 redis.conf，以及数据目录。通过 redis-server 命令指定配置文件启动

```
docker run -d --name redis4 --privileged -p 6379:6379 -v D:/Docker/etc/redis4/redis.conf:/usr/local/etc/redis/redis.conf -v D:/DockerData/redis4:/data -v D:/Docker/var/log/redis4:/var/log/redis redis:4-alpine redis-server /usr/local/etc/redis/redis.conf
```

### 启动 MySQL5.7

配置外部配置文件，以及数据目录

```
docker run -d --name mysql57 --privileged -p 3306:3306 -v D:/Docker/etc/mysql57:/etc/mysql/conf.d -v D:/Docker/var/log/mysql57:/var/log/mysql -v D:/DockerData/mysql57:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123123 -d mysql:5.7
```

### 手动构建并启动 PHP 7.2 FPM

启动，分别挂载了php.ini、php-fpm.conf、www.conf、日志目录、网站目录。

注意：网站目录需要和 Nginx 的 root 路径保持一致。

```
cd app/alpine/php72
docker build -t local/php:7.2-fpm-alpine .
docker run -d --name php72 --privileged -p 9000:9000 -v D:/Docker/etc/php72/php.ini:/usr/local/etc/php.ini -v D:/Docker/etc/php72/php-fpm.conf:/usr/local/etc/php-fpm.conf -v D:/Docker/etc/php72/php-fpm.d/www.conf:/usr/local/etc/php-fpm.d/www.conf -v D:/Docker/var/log/php72:/var/log/php -v D:/DockerData/www:/usr/share/nginx/html local/php:7.2-fpm-alpine
```

### 手动构建并启动 PHP 5.6 FPM

启动，分别挂载了php.ini、php-fpm.conf、www.conf、日志目录、网站目录。

注意：网站目录需要和 Nginx 的 root 路径保持一致。

```
cd app/alpine/php56
docker build -t local/php:5.6-fpm-alpine .
docker run -d --name php56 --privileged -p 9000:9000 -v D:/Docker/etc/php56/php.ini:/usr/local/etc/php.ini -v D:/Docker/etc/php56/php-fpm.conf:/usr/local/etc/php-fpm.conf -v D:/Docker/etc/php56/php-fpm.d/www.conf:/usr/local/etc/php-fpm.d/www.conf -v D:/Docker/var/log/php56:/var/log/php -v D:/DockerData/www:/usr/share/nginx/html local/php:5.6-fpm-alpine
```
