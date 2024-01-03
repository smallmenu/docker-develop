# Rabbitmq

mkdir /data/rabbitmq37

docker pull rabbitmq:3.7-management-alpine

docker run -d --name rabbitmq37 --restart=always --privileged -p 15672:15672 -p 5672:5672 \
-h rabbitmq37 \
-m 2G \
-v /data/rabbitmq37:/var/lib/rabbitmq \
-e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=admin  rabbitmq:3.7-management-alpine