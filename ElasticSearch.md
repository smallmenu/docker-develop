## 启动 ElasticSearch 6.8.13

单节点运行

挂载了数据目录。

```
docker pull docker.elastic.co/elasticsearch/elasticsearch:6.8.13

docker run -d --name elasticsearch6 --restart=always --privileged -p 9200:9200 -p 9300:9300 \
-v /data/elasticsearch6:/usr/share/elasticsearch/data \
-e "discovery.type=single-node" \
-e "cluster.name=docker-cluster" \
-e "node.name=docker-node" \
-e "xpack.security.enabled=true" \
-e "xpack.security.audit.enabled=true" \
-e "ELASTIC_PASSWORD=elastic" \
-e ES_JAVA_OPTS="-Xms2g -Xmx2g" \
docker.elastic.co/elasticsearch/elasticsearch:6.8.13
```

## 启动 Kibana 6.8.13

```
docker pull docker.elastic.co/kibana/kibana:6.8.13

docker run -d --name kibana6 --restart=always --privileged -p 5601:5601 \
--link elasticsearch6:elasticsearch \
-e "ELASTICSEARCH_USERNAME=elastic" \
-e "ELASTICSEARCH_PASSWORD=elastic" \
docker.elastic.co/kibana/kibana:6.8.13
```

