# 启动最新版本 Nginx
docker run -d --name nginx --privileged -p 80:80 -v D:/Docker/etc/nginx/conf.d:/etc/nginx/conf.d -v D:/Docker/etc/nginx/nginx.conf:/etc/nginx/nginx.conf -v D:/Docker/var/log/nginx:/var/log/nginx -v D:/DockerData/www:/usr/share/nginx/html nginx:alpine

# 启动 Redis4
docker run -d --name redis4 --privileged -p 6379:6379 -v D:/Docker/etc/redis4/redis.conf:/usr/local/etc/redis/redis.conf -v D:/DockerData/redis4:/data -v D:/Docker/var/log/redis4:/var/log/redis redis:4-alpine redis-server /usr/local/etc/redis/redis.conf

# 启动 MySQL5.7
docker run -d --name mysql57 --privileged -p 3306:3306 -v D:/Docker/etc/mysql57:/etc/mysql/conf.d -v D:/Docker/var/log/mysql57:/var/log/mysql -v D:/DockerData/mysql57:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123123 -d mysql:5.7

# 构建启动 PHP 7.2 FPM
cd app/alpine/php72
docker build -t local/php:7.2-fpm-alpine .
docker run -d --name php72 --privileged -p 9000:9000 -v D:/Docker/etc/php72/php.ini:/usr/local/etc/php.ini -v D:/Docker/etc/php72/php-fpm.conf:/usr/local/etc/php-fpm.conf -v D:/Docker/etc/php72/php-fpm.d/www.conf:/usr/local/etc/php-fpm.d/www.conf -v D:/Docker/var/log/php72:/var/log/php -v D:/DockerData/www:/usr/share/nginx/html local/php:7.2-fpm-alpine

# 构建启动 PHP 5.6 FPM
cd app/alpine/php56
docker build -t local/php:5.6-fpm-alpine .
docker run -d --name php56 --privileged -p 9000:9000 -v D:/Docker/etc/php56/php.ini:/usr/local/etc/php.ini -v D:/Docker/etc/php56/php-fpm.conf:/usr/local/etc/php-fpm.conf -v D:/Docker/etc/php56/php-fpm.d/www.conf:/usr/local/etc/php-fpm.d/www.conf -v D:/Docker/var/log/php56:/var/log/php -v D:/DockerData/www:/usr/share/nginx/html local/php:5.6-fpm-alpine