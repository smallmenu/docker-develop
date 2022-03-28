FROM php:7.3-fpm-alpine

# 替换 alpine 信源
RUN sed -i "s#dl-cdn.alpinelinux.org#mirrors.aliyun.com#g" /etc/apk/repositories

# docker-php-extension-installer 扩展
ADD https://github.com/mlocati/docker-php-extension-installer/releases/latest/download/install-php-extensions /usr/local/bin/
RUN chmod uga+x /usr/local/bin/install-php-extensions && sync
RUN install-php-extensions bcmath bz2 calendar gettext gd gmp igbinary pdo_mysql shmop sockets sysvmsg sysvsem sysvshm zip opcache tidy
RUN install-php-extensions imagick redis mcrypt 

# 工具
RUN apk add tzdata curl drill && rm -rf /var/cache/apk/*
RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && echo 'Asia/Shanghai' > /etc/timezone