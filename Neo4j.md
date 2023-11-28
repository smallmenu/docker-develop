## 启动 Neo4J 4.4

```
docker run -d --privileged --name neo4j-4.4.3 -p 7474:7474 -p 7687:7687 \
-v /data/neo4j-4.4.3/data:/data \
-v /data/neo4j-4.4.3/logs:/logs \
-v /data/neo4j-4.4.3/import:/var/lib/neo4j/import \
-v /data/neo4j-4.4.3/plugins:/plugins \
--env NEO4J_AUTH=neo4j/austria-rabbit-list-rider-prism-8926 \
--env NEO4JLABS_PLUGINS='["graph-data-science"]' \
neo4j:4.4.3
```