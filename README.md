https://github.com/deviantony/docker-elk/blob/master/README.md

```bash
$ git clone https://github.com/wangriyu/docker-elk.git
$ cd docker-elk
$ docker-compose up -d
```

localhost:9000 -> es 集群状态
localhost:5601 -> kibana 界面

修改配置:

logstash: logstash/pipeline/logstash.conf
elasticsearch: elasticsearch/conf/elasticsearch.yml
