# Docker Windows Develop Eenviroment

## 快速开始

* 安装 Docker Desktop Installer
* 使用 Linux 容器，自行进行 Docker 初始化配置，包括共享目录， docker 镜像源，网络，主机资源等
* clone 项目到 D:/Docker
* 创建数据目录 D:/DockerData

### 启动 Nginx

```
docker run -d --name nginx --privileged -p 80:80 -v D:/Docker/etc/nginx/conf.d:/etc/nginx/conf.d -v D:/Docker/etc/nginx/nginx.conf:/etc/nginx/nginx.conf -v D:/Docker/var/log/nginx:/var/log/nginx -v D:/DockerData/www:/usr/share/nginx/html nginx:alpine
```

### 启动 Redis4

```
docker run -d --name redis4 --privileged -p 6379:6379 -v D:/Docker/etc/redis4/redis.conf:/usr/local/etc/redis/redis.conf -v D:/DockerData/redis4:/data -v D:/Docker/var/log/redis4:/var/log/redis redis:4-alpine redis-server /usr/local/etc/redis/redis.conf
```

### 启动 MySQL5.7

```
docker run -d --name mysql57 --privileged -p 3306:3306 -v D:/Docker/etc/mysql57:/etc/mysql/conf.d -v D:/Docker/var/log/mysql57:/var/log/mysql -v D:/DockerData/mysql57:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123123 -d mysql:5.7
```

### 手动构建并启动 PHP 7.2 FPM

```
cd app/alpine/php72
docker build -t local/php:7.2-fpm-alpine .
docker run -d --name php72 --privileged -p 9000:9000 -v D:/Docker/etc/php72/php.ini:/usr/local/etc/php.ini -v D:/Docker/etc/php72/php-fpm.conf:/usr/local/etc/php-fpm.conf -v D:/Docker/etc/php72/php-fpm.d/www.conf:/usr/local/etc/php-fpm.d/www.conf -v D:/Docker/var/log/php72:/var/log/php -v D:/DockerData/www:/usr/share/nginx/html local/php:7.2-fpm-alpine
```

### 手动构建并启动 PHP 5.6 FPM

```
cd app/alpine/php56
docker build -t local/php:5.6-fpm-alpine .
docker run -d --name php56 --privileged -p 9000:9000 -v D:/Docker/etc/php56/php.ini:/usr/local/etc/php.ini -v D:/Docker/etc/php56/php-fpm.conf:/usr/local/etc/php-fpm.conf -v D:/Docker/etc/php56/php-fpm.d/www.conf:/usr/local/etc/php-fpm.d/www.conf -v D:/Docker/var/log/php56:/var/log/php -v D:/DockerData/www:/usr/share/nginx/html local/php:5.6-fpm-alpine
```

## 主要说明

### 网络

#### docker0

Windows 的 Docker 桌面中没有 docker0，它存在于虚拟机中

#### 容器连接主机上的服务

虽然在容器中可以直接 ping 通主机，但是现在不允许直接进行服务通讯。

可以通过 `host.docker.internal` 或 `gateway.docker.internal` 来实现通讯。

### Nginx

官方默认  debian 基础镜像

#### Alpine 镜像

nginx:alpine
nginx:<version>-alpine

#### 直接启动

默认的配置文件启动，只挂载网站目录。

```
docker run -d --name nginx --privileged -p 80:80 -v D:/Docker/data/www:/usr/share/nginx/html nginx:alpine
```

#### 配置文件启动

挂载了主配置文件，额外配置目录，日志目录，网站目录

```
docker run -d --name nginx --privileged -p 80:80 -v D:/Docker/etc/nginx/conf.d:/etc/nginx/conf.d -v D:/Docker/etc/nginx/nginx.conf:/etc/nginx/nginx.conf -v D:/Docker/var/log/nginx:/var/log/nginx -v D:/Docker/data/www:/usr/share/nginx/html nginx:alpine
```

### Redis 4 

官方默认  debian 基础镜像

#### Alpine 镜像

redis:alpine
redis:<version>-alpine

#### 直接启动

默认的配置文件启动，只挂载数据目录

```
docker run -d --name redis4 --privileged -p 6379:6379 -v D:/Docker/data/redis4:/data redis:4-alpine
```

#### 配置文件启动

配置外部配置文件 redis.conf，以及数据目录。通过 redis-server 命令指定配置文件启动

注意配置文件需要 `daemonize no`，否则 docker 探测不到应用状态无法启动

```
docker run -d --name redis4 --privileged -p 6379:6379 -v D:/Docker/etc/redis4/redis.conf:/usr/local/etc/redis/redis.conf -v D:/Docker/data/redis4:/data -v D:/Docker/var/log/redis4:/var/log/redis redis:4-alpine redis-server /usr/local/etc/redis/redis.conf
```


### PHP

官方默认 debian 基础镜像

#### Alpine 镜像

php:<version>-cli-alpine
php:<version>-fpm-alpine

#### 直接启动

```
docker run -d --name php72 --privileged -p 9000:9000 php:7.2-fpm-alpine
```

#### docker-php-extension-installer


#### 本地构建

因为 PHP 会涉及到很多扩展安装，一般我们需要本地构建镜像。借助 install-php-extensions 脚本可以方便的安装扩展依赖。

参考：

https://github.com/mlocati/docker-php-extension-installer

Dockerfile

```
FROM php:7.2-fpm-alpine

# 替换 alpine 信源
RUN sed -i "s#dl-cdn.alpinelinux.org#mirrors.aliyun.com#g" /etc/apk/repositories

# docker-php-extension-installer 扩展
ADD https://raw.githubusercontent.com/mlocati/docker-php-extension-installer/master/install-php-extensions /usr/local/bin/
RUN chmod uga+x /usr/local/bin/install-php-extensions && sync
RUN install-php-extensions bcmath gd calendar igbinary redis memcache mongodb pdo_mysql zip opcache tidy grpc protobuf rdkafka 

RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && echo 'Asia/Shanghai' > /etc/timezone
```

本地构建：

```
docker build -t local/php:7.2-fpm-alpine .
```

启动，分别挂载了php.ini、php-fpm.conf、www.conf、日志目录、网站目录。

注意：网站目录需要和 Nginx 的 root 路径保持一致。

```
docker run -d --name php72 --privileged -p 9000:9000 -v D:/Docker/etc/php72/php.ini:/usr/local/etc/php.ini -v D:/Docker/etc/php72/php-fpm.conf:/usr/local/etc/php-fpm.conf -v D:/Docker/etc/php72/php-fpm.d/www.conf:/usr/local/etc/php-fpm.d/www.conf -v D:/Docker/var/log/php72:/var/log/php -v D:/Docker/data/www:/usr/share/nginx/html local/php:7.2-fpm-alpine
```

Nginx 转发 PHP-FPM，需要使用 host.docker.internal 这个地址，表示转发到宿主机上。

```
location ~ \.php$
{
    fastcgi_pass  host.docker.internal:9000;
    fastcgi_index index.php;
    include fastcgi.conf;
}
```

### MySQL

官方默认 Debian 基础镜像

#### 直接启动

默认配置文件，只挂载数据目录。如果已存在的数据目录会忽略 MYSQL_ROOT_PASSWORD

```
docker run -d --name mysql57 --privileged -p 3306:3306 -v D:/Docker/data/mysql57:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123123 -d mysql:5.7
```

#### 配置文件启动

配置外部配置文件 redis.conf，以及数据目录

```
docker run -d --name mysql57 --privileged -p 3306:3306 
-v D:/Docker/etc/mysql57:/etc/mysql/conf.d -v D:/Docker/var/log/mysql57:/var/log/mysql -v D:/Docker/data/mysql57:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123123 -d mysql:5.7
```

### RabbitMQ

management 是自带安装了管理 UI 工具的镜像

#### Alpine 镜像

rabbitmq:<version>-management
rabbitmq:<version>-management-alpine

#### 直接启动


默认配置文件

```
docker run -d --name rabbitmq --privileged -p 15672:15672 -p 5672:5672 -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=admin  rabbitmq:3.7-management-alpine
```
