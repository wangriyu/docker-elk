日志流:

![ELK-Stack](https://src.wangriyu.wang/images/blog/ELK/ELK-Stack.svg)

Filebeat 抓取的非结构化日志格式:

```log
Mon Sep 10 2018 12:06:16 GMT+0800 (CST) - info: type: SyncSessionRangeService  uid:9182093812301298392103{}

18-09-07 08:19:43 - info: GRPC REQUEST & RESPONSE{
		"uid": 1290381298494,
		"type": "SyncSessionRangeService",
		"data": {
			"session_id": 123456,
			"session_type": 2,
			"start_id": 0,
			"stop_id": 3
		},
		"localId": 349058034598,
		"result": {
			"type": 5,
			"is_sync": true,
			"is_ack": true,
			"require_ack": false,
			"error": 0,
			"continue": false,
			"sync_local_id": 0,
			"messages": []
		}
	}

18-09-06 22:07:33 - info: type: SyncSessionRangeService  uid:20373933{}

18-09-07 08:19:43 - info: GRPC REQUEST & RESPONSE{
			"uid": 5792715,
			"type": "SyncSessionRangeService",
			"data": {
				"session_id": 13416794,
				"session_type": 2,
				"start_id": 0,
				"stop_id": 3
			},
			"localId": 1536279501,
			"result": {
				"type": 5,
				"is_sync": true,
				"is_ack": true,
				"require_ack": false,
				"error": 0,
				"continue": false,
				"sync_local_id": 0,
				"messages": []
			}
		}

[08/Sep/2018:01:49:55 +0800] check photo https://sgchatfiles.bldimg.com/2018/9/8/1/49/16263641_1536342594629.jpg

[08/Sep/2018:01:49:55 +0800] check "result": {
    "type": 5,
    "is_sync": true,
    "is_ack": true,
    "require_ack": false,
    "error": 0,
    "continue": false,
    "sync_local_id": 0,
    "messages": []
}
```

filebeat.yml:

```yaml
filebeat.inputs:
  - type: log
    enabled: true
    paths:
      - /var/log/req/req-worker-*.log # 文件名和目录名都可以用通配符，不过 path/*/*.log 不包括 path 根目录的文件
    fields: # 添加额外的字段
      log_topic: "log-test"
      service: "backend-req"
    fields_under_root: true # field 字段会放在根索引下，否则会放在 fields 字段下
    ignore_older: 24h # 忽略 24 小时前的文件
    scan_frequency: 11s # 设置不同的时间，这样可以错开扫描高峰
    max_backoff: 11s
    backoff: 11s
    harvester_buffer_size: 51200 # 采集的 buffer 大小
    close_timeout: 1h # 因为我这里的文件是一个小时产生一个，所以直接设置采集器默认 1 小时关闭
    clean_inactive: 25h # 需要大于 ignore_older + scan_frequency
    harvester_limit: 10 # 限制最多爬取个数，默认不限制，如果碰到文件数很多初次扫描会占用大量 cpu
  - type: log
    enabled: true
    paths:
      - /var/log/push/push-worker.log.*
    fields:
      log_topic: "log-test"
      service: "backend-push"
    fields_under_root: true
    ignore_older: 24h
    scan_frequency: 19s
    max_backoff: 19s
    backoff: 19s
    harvester_buffer_size: 51200
    close_timeout: 1h
    clean_inactive: 25h
    harvester_limit: 10
  - type: log
    enabled: true
    paths:
      - /var/log/send/send-worker.log.*
    fields:
      log_topic: "log-test"
      service: "backend-send"
    fields_under_root: true
    ignore_older: 24h
    scan_frequency: 13s
    max_backoff: 13s
    backoff: 13s
    harvester_buffer_size: 51200
    close_timeout: 1h
    clean_inactive: 25h
    harvester_limit: 10
  - type: log
    enabled: true
    paths:
      - /var/log/sync/sync-worker.log.*
    fields:
      log_topic: "log-test"
      service: "backend-sync"
    fields_under_root: true
    ignore_older: 24h
    scan_frequency: 17s
    max_backoff: 17s
    backoff: 17s
    harvester_buffer_size: 51200
    close_timeout: 1h
    clean_inactive: 25h
    harvester_limit: 10
  - type: log
    enabled: true
    paths:
      - /var/log/delete/delete-worker.log.*
    fields:
      log_topic: "log-test"
      service: "backend-delete"
    fields_under_root: true
    ignore_older: 24h
    scan_frequency: 71s
    max_backoff: 71s
    backoff: 71s
    harvester_buffer_size: 51200
    close_timeout: 1h
    clean_inactive: 25h
    harvester_limit: 10
  - type: log
    enabled: true
    paths:
        - /var/log/php/error.log
    fields:
        log_topic: "log-im"
        service: "backend-php-error"
    fields_under_root: true
    ignore_older: 24h
    scan_frequency: 23s
    max_backoff: 23s
    backoff: 23s
    clean_inactive: 25h
    close_renamed: true # 因为这个日志文件会定期重命名并压缩，所以设置 close_renamed 可以关闭采集器
    multiline.pattern: '^\[[0-9]{2}-[A-Z]{1}[a-z]{2}-[0-9]{4}' # 合并多行日志，将类似 [06-Sep-2018 ... 开头的日志向后合并
    multiline.negate: true
    multiline.match: after
    harvester_limit: 10

output.kafka:
  enabled: true
  hosts: ["10.10.X.A:9092", "10.10.X.B:9092", "10.10.X.C:9092"]
  topic: '%{[log_topic]}'
  codec.format:
    string: '%{[beat][hostname]} %{[service]} %{[message]}' # 传给 logstash 时可以用 grok 过滤插件设置 `match => { "message" => "^%{DATA:hostname} %{DATA:service} (?<message>.*)"}` 解析
    # 如果设成 ‘%{[message]}’，则是原样转发消息
  partition.round_robin:
    reachable_only: false
  required_acks: 1
  compression: none # 默认是 gzip，如果像节省开销可以设成 none
  bulk_max_size: 100
  max_message_bytes: 1000000 # 不要超过 kafka server 端设置的 message.max.size，否则超过的部分会被丢弃

output.file:
  enabled: false
  path: /home/ubuntu/test-log
  filename: output.log
  permissions: 0644
  codec.format:
    string: '%{[message]}'
```

对于多行日志，使用 pattern 处理: https://www.elastic.co/guide/en/beats/filebeat/current/multiline-examples.html

比如:
- Mon Sep 10 2018 12:06:16 GMT+0800 (CST) -> '[T|M|W|S|F]{1}[a-z]{2} [J|F|M|A|S|O|N|D]{1}[a-z]{2} [0-9]{2} [0-9]{4} [0-9]{2}:[0-9]{2}:[0-9]{2} [GMT+]{4}[0-9]{4} [(CST)]{5}'
- [06-Sep-2018 ... -> '^\[[0-9]{2}-[A-Z]{1}[a-z]{2}-[0-9]{4}'
- 18-09-07 08:19:43 -> '^[0-9]{2}-[0-9]{2}-[0-9]{2} [0-9]{2}:[0-9]{2}:[0-9]{2}'

logstash.conf:

```conf
input {
    kafka {
        bootstrap_servers => "10.10.X.A:9092,10.10.X.B:9092,10.10.X.C:9092"
        consumer_threads => 6 ## 设置成 partition 数量即可
        decorate_events => false
        topics => ["log-im"]
        group_id => "group-log-im"
        type => "log-im"
    }
    kafka {
        bootstrap_servers => "10.9.X.A:9092,10.9.X.B:9092,10.9.X.C:9092"
        consumer_threads => 8
        decorate_events => false
        topics => ["log-dev"]
        group_id => "group-log-dev"
        type => "log-dev"
    }
}

## Add your filters / logstash plugins configuration here
filter {
    mutate {
        remove_field => [ "@version" ]
    }
    if [type] == "log-im" {
        ## grok 插件可以将非结构化日志解析成结构化的
        ## 测试工具: http://grokdebug.herokuapp.com
        ## 推荐阅读: https://www.elastic.co/blog/do-you-grok-grok
        grok {
            match => { "message" => "^%{DATA:hostname} %{DATA:service} (?<message>.*)" }
            overwrite => [ "message" ]
        }
        grok {
            match => {
                "message" => [
                    "^\[%{DATA:time}\] (?<message>.*)",
                    "^(?<time>[T|M|W|S|F]{1}[a-z]{2} [J|F|M|A|S|O|N|D]{1}[a-z]{2} [0-9]{2} [0-9]{4} [0-9]{2}:[0-9]{2}:[0-9]{2} [GMT+]{4}[0-9]{4} [(CST)]{5}) - %{LOGLEVEL:level}: (?<message>.*)",
                    "^(?<time>[0-9]{2}-[0-9]{2}-[0-9]{2} [0-9]{2}:[0-9]{2}:[0-9]{2}) - %{LOGLEVEL:level}: (?<message>.*)",
                    "^(?<message>.*)"
                ]
            }
            overwrite => [ "message" ]
        }
        ## 解析 json 格式的字段
        json {
            source => "message"
            skip_on_invalid_json => true
            remove_field => [ "message" ]
        }
        ## 将日志内的时间戳解析到 @timestamp 中，方便之后 Kibana 按时间索引排序
        date {
            match => [ "time", "ISO8601", "EEE MMM dd yyyy HH:mm:ss 'GMT'Z '(CST)'", "dd/MMM/yyyy:HH:mm:ss Z", "yy-MM-dd HH:mm:ss", "dd-MMM-yyyy HH:mm:ss ZZZ", "yyyy-MM-dd HH:mm:ss ZZ", "yyyy-MM-dd HH:mm ZZ" ]
            remove_field => [ "time" ]
        }
        if [service] == "backend-php-error" {
            mutate {
                add_field => { "level" => "error" }
            }
        }
    }
}

output {
    ## 按 type 区分不同 topic 的日志，然后写入不同的目录
    if [type] == "log-im" {
        elasticsearch {
            hosts => ["localhost:9200", "10.42.X.B:9200", "10.42.X.C:9200"]
            index => "%{type}-%{+YYYY.MM.dd}"
        }
    }
    if [type] == "log-dev" {
         elasticsearch {
             hosts => ["localhost:9200", "10.42.X.B:9200", "10.42.X.C:9200"]
             index => "%{type}-%{+YYYY.MM.dd}"
        }
    }
}
```