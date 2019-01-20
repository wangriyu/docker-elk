### 使用

> https://github.com/deviantony/docker-elk/blob/master/README.md

```bash
$ git clone https://github.com/wangriyu/docker-elk.git
$ cd docker-elk
$ ELK_HOST="YourHostIP" ELK_NODENAME="es-node-01" ELK_CLUSTERNAME="elk-cluster" docker-compose -f docker-compose.yml up -d
## add docker log
# ELK_HOST="YourHostIP" ELK_NODENAME="es-node-01" ELK_CLUSTERNAME="elk-cluster" docker-compose -f docker-compose.yml -f extensions/logspout/logspout-compose.yml up -d
```

### test log
```bash
$ echo '{"project": "docker-elk"}' > /dev/udp/localhost/5000
```

### WebUI
- http://YourHostIP:9000/#/overview?host=http:%2F%2Flocalhost:9200 -> es 集群状态
- YourHostIP:5601 -> kibana 界面

### 修改配置

logstash: logstash/pipeline/logstash.conf

elasticsearch: elasticsearch/conf/elasticsearch.yml

kibana: kibana/conf/kibana.yml

### 参考配置

- elasticsearch-reference.md
- logstash-reference.md
