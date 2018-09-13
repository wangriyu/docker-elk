> [重要配置的修改 - Elasticsearch: 权威指南](https://www.elastic.co/guide/cn/elasticsearch/guide/current/important-configuration-changes.html)

```yaml
## ---------------------------------- Cluster -----------------------------------
## Use a descriptive name for your cluster:

## 集群名称，用于定义哪些 elasticsearch 节点属同一个集群。
cluster.name: elk-cluster

## ------------------------------------ Node ------------------------------------

## 节点名称，用于唯一标识节点，不可重名，可以设置 ${HOSTNAME}
node.name: elk-node-1

## https://www.elastic.co/guide/en/elasticsearch/reference/6.3/modules-node.html
## 默认 master、data、injest 都为 true，此外还有一种 tribe node 用于跨集群操作
## 1、以下列出了三种集群拓扑模式，如下:
## 如果想让节点不具备选举主节点的资格，只用来做数据存储节点:
# node.master: false
# node.data: true

## 2、如果想让节点成为主节点，且不存储任何数据，只作为集群协调者。
# node.master: true
# node.data: false

## 3、如果想让节点既不成为主节点,又不成为数据节点,那么可将他作为搜索器,从节点中获取数据,生成搜索结果等
# node.master: false
# node.data: false
# node.injest: true

## 这个配置限制了单机上可以开启的 ES 存储实例的个数，当我们需要单机多实例，则需要把这个配置赋值 2，或者更高。
# node.max_local_storage_nodes: 1

## ----------------------------------- Index ------------------------------------

## 设置索引的分片数，默认为 5，"number_of_shards" 是索引创建后一次生成的，后续不可更改设置
index.number_of_shards: 5

## 设置索引的副本数，默认为 1
index.number_of_replicas: 1

## ----------------------------------- Paths ------------------------------------

## 数据存储路径，可以设置多个路径用逗号分隔，有助于提高 IO。 # path.data: /home/path1,/home/path2
path.data: /home/elk/server3_data

## 日志文件路径
path.logs: /var/log/elasticsearch

## 临时文件的路径
path.work: /path/to/work

## ----------------------------------- Memory -------------------------------------

## 确保 ES_MIN_MEM 和 ES_MAX_MEM 环境变量设置为相同的值，以及机器有足够的内存分配给 Elasticsearch
## 注意:内存也不是越大越好，一般 64 位机器最大分配内存不要超过 32G

## 当 JVM 开始写入交换空间时（swapping）ElasticSearch 性能会低下，最好按 https://www.elastic.co/guide/en/elasticsearch/reference/6.3/setup-configuration-memory.html 关闭交换
## 设置这个属性为 true 来锁定内存，同时也要允许 elasticsearch 的进程可以锁住内存，linux下可以通过 `ulimit -l unlimited` 命令
bootstrap.memory_lock: true

## 节点用于 fielddata 的最大内存，如果 fielddata 
## 达到该阈值，就会把旧数据交换出去。该参数可以设置百分比或者绝对值。默认设置是不限制，所以强烈建议设置该值，比如 10%。
indices.fielddata.cache.size: 20%

# indices.fielddata.cache.expire  这个参数绝对绝对不要设置！

## 默认值是JVM堆内存的 60%,注意为了让设置正常生效，一定要确保 indices.breaker.fielddata.limit 的值大于 indices.fielddata.cache.size 的值。否则的话，fielddata 大小一到 limit 阈值就报错，就永远道不了 size 阈值，无法触发对旧数据的交换任务了。
indices.breaker.fielddata.limit: 60%

## ------------------------------------ Network And HTTP -----------------------------

## 设置绑定的 ip 地址，可以是 ipv4 或 ipv6 的， 默认为 0.0.0.0
network.bind_host: 192.168.0.1

## 设置其它节点和该节点通信的 ip 地址，如果不设置它会自动设置，值必须是个真实的 ip 地址
network.publish_host: 192.168.0.1

## 同时设置 bind_host 和 publish_host 上面两个参数
network.host: 192.168.0.1

## 设置集群中节点间通信的 tcp 端口，默认是 9300
transport.tcp.port: 9300

## 设置是否压缩 tcp 传输时的数据，默认为 false,不压缩
transport.tcp.compress: false

## 设置对外服务的 http 端口，默认为 9200
http.port: 9200

## 设置请求内容的最大容量，默认 100mb
http.max_content_length: 100mb

## ------------------------------------ Translog -------------------------------------

## 当事务日志累积到多少条数据后flush一次。
index.translog.flush_threshold_ops: 50000

## --------------------------------- Discovery --------------------------------------

## https://www.elastic.co/guide/en/elasticsearch/reference/6.3/modules-node.html#split-brain
## 这个参数决定了要选举一个 Master 至少需要多少个节点，默认值是 1，推荐设置为 N/2 + 1，N 是集群中 master 候选节点的数量，这样可以有效避免脑裂
discovery.zen.minimum_master_nodes: 1

## 在 java 里面 GC 是很常见的，但在 GC 时间比较长的时候。在默认配置下，节点会频繁失联。节点的失联又会导致数据频繁重传，甚至会导致整个集群基本不可用。

## discovery参数是用来做集群之间节点通信的，默认超时时间是比较小的。我们把参数适当调大，避免集群 GC 时间较长导致节点的丢失、失联。
discovery.zen.fd.ping_timeout: 200s
discovery.zen.fd.ping.interval: 30s
discovery.zen.fd.ping.retries: 6

## 设置集群中节点的探测列表，新加入集群的节点需要加入列表中才能被探测到
discovery.zen.ping.unicast.hosts: ["10.10.X.A:9300","10.10.X.B:9300",10.10.X.C:9300]

## --------------------------------- Gateway --------------------------------------

## https://www.elastic.co/guide/en/elasticsearch/reference/6.3/modules-gateway.html
gateway.recover_after_nodes: 2
gateway.expected_nodes: 3
gateway.recover_after_time: 5m

indices.store.throttle.type: merge
indices.store.throttle.max_bytes_per_sec: 100mb
```