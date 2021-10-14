

elasticsearch7.2实操

## 安装篇：
#### 系统参数设置

/etc/sysctl.conf
```
vm.max_map_count = 655360  #加大虚拟内存
vm.swappiness = 1	# 关闭交换内存
```
执行以下命令，确保生效配置生效：sysctl -p

/etc/security/limits.conf
```
* soft nofile 655360		#加大打开文件数
* hard nofile 655360		#加大打开文件数
* soft nproc unlimited     #创建线程数
* hard nproc unlimited     #创建线程数
```

Java环境变量
tar -xvf jdk-8u131-linux-x64.tar.gz

```
vi  /etc/profile

export JAVA_HOME=/home/es/jdk1.8.0_201
export PATH=$PATH:$JAVA_HOME/bin

source /etc/profile
java -version
```


#### liunx新建es用户和组

```
groupadd es

useradd es -g es -p es
```



### 安装elasticsearch

1.切换到非root用户

1.下载elasticsearch-7.2.0

wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.2.0-linux-x86_64.tar.gz

tar -xvf elasticsearch-7.2.0-linux-x86_64.tar.gz

修改目录名称 

```
mv elasticsearch-7.2.0 elasticsearch-7.2.0-9202
```

vi elasticsearch-7.2.0-9202/config/elasticsearch.yml   #替换ip即可

cluster.name: elasticsearch_cluster
node.name: es_192.168.199.101_9202
http.port: 9202
transport.tcp.port: 9302
node.master: true
node.data: true
discovery.zen.ping.unicast.hosts: ["192.168.199.101:9302", "192.168.199.101:9303", "192.168.199.101:9304"] 
cluster.initial_master_nodes: ["es_192.168.199.101_9202","es_192.168.199.101_9203","es_192.168.199.101_9204"]
#防止脑裂这个参数控制的是，一个节点需要看到的具有master节点资格的最小数量，然后才能在集群中做操作。官方的推荐值是(N/2)+1，其中N是具有master资格的节点的数量
discovery.zen.minimum_master_nodes: 2
#浏览器跨域相关
http.cors.enabled: true
http.cors.allow-headers: Authorization,X-Requested-With,Content-Length,Content-Type
http.cors.allow-origin: "*"
network.host: 0.0.0.0

修改jvm配置参数 
vi config/jvm.options
-Xms512m
-Xmx512m

复制目录到其它节点

```
cp -r elasticsearch-7.2.0-9202 elasticsearch-7.2.0-9203
cp -r elasticsearch-7.2.0-9202 elasticsearch-7.2.0-9204
```

修改elasticsearch-7.2.0-9203和elasticsearch-7.2.0-9204目录内配置文件端口

```
vi elasticsearch-7.2.0-9203/config/elasticsearch.yml  #修改以下三个属性

node.name: es_192.168.199.101_9203
http.port: 9203
transport.tcp.port: 9303

vi elasticsearch-7.2.0-9204/config/elasticsearch.yml  #修改以下三个属性

node.name: es_192.168.199.101_9204
http.port: 9204
transport.tcp.port: 9304
```



//启动
bin/elasticsearch -d

curl http://localhost:9202/



更改elasticsearch-7.2.0文件夹及内部文件的所属用户及组命令

```
chown -R es:es elasticsearch-7.2.0
```




### 安装kibnan
kibnan-7.2.0插件安装  es用户

1.下载kibnan-7.2.0

wget https://artifacts.elastic.co/downloads/kibana/kibana-7.2.0-linux-x86_64.tar.gz

tar -xvf kibana-7.2.0-linux-x86_64.tar.gz

关联ES实例

cd kibana-7.2.0-linux-x86_64

vi  config/kibana.yml  

server.host: "0.0.0.0"     
server.port: 5601

elasticsearch.hosts: ["http://localhost:9202"]

i18n.locale: "zh-CN"  #汉化

启动kibana

bin/kibana  

后台启动nohup bin/kibana &

页面访问kibnan

http://192.168.199.101:5601/

 停止kibana
 netstat -tunlp|grep 5601 #通过端口查找到进程
 tcp        0      0 0.0.0.0:5601            0.0.0.0:*               LISTEN      692828/node 
 kill 692828



#### 离线安装Node.js

```
tar -zxvf nodejs.tar.gz 
ln -s /home/es/soft/nodejs/bin/node /usr/local/bin/node
ln -s /home/es/soft/nodejs/bin/npm /usr/local/bin/npm
ln -s /home/es/soft/nodejs/bin/grunt  /usr/local/bin/grunt 
node -v
npm -v
grunt -version
```



### 安装head
head插件安装  root用户
解压
tar -zxvf elasticsearch-head.tar.gz
修改配置文件
vi elasticsearch-head/Gruntfile.js
启动
nohup grunt server &

访问http://192.168.199.101:9102/ 
内连接http://192.168.199.101:9202/



## 基本概念：
官方文档 
https://www.elastic.co/guide/index.html
https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html
Elasticsearch: 权威指南 中文版
https://www.elastic.co/guide/cn/elasticsearch/guide/current/index.html

### 查看集群和节点信息
_cat/health
_cat/nodes


### 分片和副本

索引分片数量设置:
索引分片数过少或过多都不合适。分片数过多会对集群造成额外的压力。而分片数过少会导至单个分片索引过大，检索速度慢。建议单个分片的数据大小10G-50G, 一般存储20G左右的索引数据，所以，分片数量=数据总量/20G
节点分片数量设置:
每GB堆内存不应该超过20个分片，因此单节点30G分片数量不应该超过600个。
副本数设置:
副本默认1个即可，副本越多虽然可以提升搜索的能力，但是会对集群造成很大压力

https://www.elastic.co/guide/en/elasticsearch/reference/current/size-your-shards.html

~~~
#分片测试
#默认1个分片，1个副本
PUT /shard_test
#5个分片，默认1个副本
PUT /shard_test1
{
  "settings":{
  "number_of_shards":5
  }
}
#5个分片，2个副本 设置2个副本但是只有2个节点，所以有副本未分配
PUT /shard_test2
{
  "settings":{
  "number_of_shards":5,
  "number_of_replicas":2
  }
}
PUT /shard_test2/_settings
{
 "number_of_replicas":0
}
#查看分片数
#主分片和副本不会在同一个节点上
GET /shard_test1/_search_shards
DELETE /shard_test1



~~~





### 映射Mapping

Mapping,就是对索引库中索引的字段名称及其数据类型进行定义，类似于关系数
据库中表建立时要定义字段名及其数据类型那样，它可以动态添加字段。也可以不指定
mapping，因为es会自动根据数据格式定义它的类型，如果你需要对某些
字段添加特殊属性（如：定义使用其它分词器、是否分词），就必须
手动添加mapping

```
#定义映射 字段类型
POST /shard_test2/_mapping
{
  "properties": {
    "name": { "type": "keyword"},
    "age": { "type": "long"},
    "birth_date": { "type": "date"}
  }
}
#查看映射
GET /shard_test2/_mapping
```


### 基本操作
创建索引
开关索引
查看索引、类型、字段、数据信息
执行基本查询
执行复合查询（增删改等）

~~~
#写入
PUT /megacorp/_doc/1
{
    "first_name" :  "John",
    "last_name" :   "Smith",
    "age" :         25,
    "about":        "I love to go rock climbing",
    "interests":  [ "sport","music" ]
}
PUT /megacorp/_doc/2
{
    "first_name" :  "Jane",
    "last_name" :   "Smith",
    "age" :         32,
    "about" :       "I like to collect rock albums",
    "interests":  [ "music" ]
}
PUT /megacorp/_doc/3
{
    "first_name" :  "Douglas",
    "last_name" :   "Fir",
    "age" :         35,
    "about":        "I like to build cabinets",
    "interests":  [ "forestry" ]
}
#查看索引信息
GET /megacorp
#按ID查询员工
GET /megacorp/_doc/1
#修改
PUT /megacorp/_doc/1
{
    "first_name" :  "John",
    "last_name" :   "Smith",
    "age" :         26,
    "about":        "I love to go rock climbing",
    "interests":  [ "sport","music" ]
}
#删除文档
DELETE /megacorp/_doc/1
#无条件搜索
GET /megacorp/_search  
#简单条件搜索
GET /megacorp/_search?q=last_name:"Smith"
#DSL查询表达式搜索
#全文搜索
GET /megacorp/_search
{
  "query": {
    "match": {
    "about": "rock climbing"
  }}
}
#短语搜索 
GET /megacorp/_search
{
  "query": {
    "match_phrase": {
    "about": "rock climbing"
  }}
}
#模糊搜索
#SELECT * FROM megacorp WHERE about like "%like%"
GET /megacorp/_search
{
  "query": {
    "wildcard": {
    "about": "rock"
  }}
}
#完全匹配
#SELECT * FROM megacorp WHERE last_name = 'Smith'
GET /megacorp/_search
{
  "query": {
    "term": {
    "last_name.keyword": "Smith"
  }}
}
#过滤查询
#must: 条件必须满足，相当于 and
#should: 条件可以满足也可以不满足，相当于 or
#must_not: 条件不需要满足，相当于 not
#SELECT * FROM megacorp WHERE (last_name = "Smith"  and first_name != "John") 
GET /megacorp/_search
{
  "query": {
    "bool": {
      "must": [{"term": {
        "last_name.keyword": {
          "value": "Smith"
        }
      }}],
      "must_not": [{"term": {
        "first_name.keyword": {
          "value": "John"
        }
      }}]
     
    }
  }
}
#SELECT * FROM megacorp WHERE (age = 25 OR age = 32) 
GET /megacorp/_search
{
  "query": {
    "bool": {
      "must": [],
      "must_not": [],
      "should": [
        {
          "term": {
            "age": 25
          }
        },
        {
          "term": {
            "age": 32
          }
        }
      ]
    }
  }
}
#range范围过滤
#gt :  > 大于
#lt :  < 小于
#gte :  >= 大于等于
#lte :  <= 小于等于
#SELECT * FROM megacorp WHERE age >= 20 AND age < 30
GET /megacorp/_search
{
  "query": {
    "bool": {
            "must": [
        {
          "range": {
            "age": {
              "gt": 20,
              "lt": 30
            }
          }
        }
      ]
    }
  }
}
#聚合
GET /megacorp/_search
{
  "aggs": {
    "all_interests": {
      "terms": {"field":"interests.keyword"}
    }
  }  
}
#先查询再聚合
GET /megacorp/_search
{
  "query": {"match": {
    "last_name": "Smith"
  }}, 
  "aggs": {
    "all_interests": {
      "terms": {"field":"interests.keyword"}
    }
  }  
}
#嵌套聚合
GET /megacorp/_search
{
  "aggs": {
    "all_interests": {
      "terms": {"field":"interests.keyword"},
      "aggs":{
        "avg_age":{
          "avg":{"field":"age"}
        }
      }
    }
  }  
}
~~~


### 分词

下载ik分词插件

https://github.com/medcl/elasticsearch-analysis-ik

目前最新版

https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.2.0/elasticsearch-analysis-ik-7.2.0.zip

使用自带的标准分词，每个汉字都会分词

离线安装ik，每个节点需要安装,安装完重启

bin/elasticsearch-plugin install -b file:/home/ivory/es7.2.0download/elasticsearch-analysis-ik-7.2.0.zip

ik_smart: 会做最粗粒度的拆分，比如会将“中华人民共和国国歌”拆分为“中华人民共和国,国歌”，适合 Phrase 查询。

ik_max_word: 会将文本做最细粒度的拆分，比如会将“中华人民共和国国歌”拆分为“中华人民共和国,中华人民,中华,华人,人民共和国,人民,人,民,共和国,共和,和,国国,国歌”，会穷尽各种可能的组合，适合 Term Query；

~~~
POST /_analyze
{
  "analyzer": "standard",
  "text": "中华人民共和国国歌"
}
POST /_analyze
{
  "analyzer": "ik_smart",
  "text": "中华人民共和国国歌"
}
POST /_analyze
{
  "analyzer": "ik_max_word",
  "text": "中华人民共和国国歌"
}
~~~

### 分片内部原理
```
#refresh操作可以通过API设置：
#当我们进行大规模的创建索引操作的时候，最好将将refresh关闭。
#修改定时刷新时间
PUT /megacorp/_settings
{
  "refresh_interval": "10s"
}
#-1 关闭定时刷新时间,由集群自动刷新
PUT /megacorp/_settings
{
  "refresh_interval": -1
}

#手动刷新
# 刷新所有索引
POST /_refresh
# 指定刷新索引
POST /megacorp/_refresh

#手动刷新translog文件
POST /_flush


#强制段合并
POST /_forcemerge
#强制某个索引制段合并  max_num_segments最大段数量
POST /megacorp/_forcemerge?max_num_segments=2

```
## 性能优化系统参数配置

1./etc/sysctl.conf #增加以下参数  
```
vm.max_map_count=655360   #虚拟内存
vm.swappiness = 1	#永久关闭内存交换或设置为1
```

执行以下命令，确保生效配置生效：
sysctl -p
https://www.elastic.co/guide/cn/elasticsearch/guide/current/_file_descriptors_and_mmap.html
https://www.elastic.co/guide/cn/elasticsearch/guide/current/heap-sizing.html


2. /etc/security/limits.conf
```
* soft nofile 655360
* hard nofile 655360
* soft nproc unlimited     #创建线程数
* hard nproc unlimited     #创建线程数
* soft memlock unlimited   #锁定内存不限制
* hard memlock unlimited   #锁定内存不限制
```
3. 如果上述配置没生效，设置/etc/security/limits.d/20-nproc.conf
```
* soft nproc 655360
```
https://www.elastic.co/guide/en/elasticsearch/reference/current/max-number-threads-check.html#max-number-threads-check

## 监控：
### 开启收集监控数据
```
PUT /_cluster/settings
{
  "persistent": {
    "xpack.monitoring.collection.enabled":true
  }
}
GET /_cluster/settings
```
### 开启慢日志监控
```
PUT /_settings
{
    "index.search.slowlog.threshold.query.warn" : "10s", 
    "index.search.slowlog.threshold.fetch.debug": "300ms", 
    "index.indexing.slowlog.threshold.index.info": "5s" 
}
GET /_settings
```


## 运维：


### 日常重启

### 滚动重启：适应于日常维护升级

1. 可能的话，停止写入新的数据，停止写入可以帮助后续提高恢复速度。停止写入数据后, 将数据刷到磁盘，执行命令如下，可能需要几分钟，等待完成后再执行后续步骤。
```
curl -u admin: admin -XPOST http:// localhost:9200/_flush/synced
```
2. 临时禁止分片分配，执行命令如下：
```
curl -u admin:admin -XPUT localhost:9200/_cluster/settings -H 'Content-Type: application/json' -d '{"transient":{"cluster.routing.allocation.enable" : "none"}}'
```
3. 关闭单个节点。
4. 执行维护/升级。
5. 重启节点，然后确认它加入到集群了。
```
_cat/health; _cat/nodes
```

6. 用如下命令重启分片分配：
```
curl -u admin:admin -XPUT localhost:9200/_cluster/settings -H 'Content-Type: application/json' -d '{"transient":{"cluster.routing.allocation.enable" : "all"}}'
```
分片再平衡会花一些时间。一直等到集群变成 green 绿色 状态后再继续。

7. 重复第 2 到 6 步操作剩余节点。
8. 所有节点重启完成并green, 再次开始写入。

 

### 全量重启：适应于故障恢复

1.永久禁止分片移动

```
curl  -XPUT localhost:9206/_cluster/settings -H 'Content-Type: application/json' -d'{"persistent":{"cluster.routing.allocation.enable" : "none"}}'
```

2.内存数据写到磁盘

```
curl  -XPOST localhost:9206/_flush/synced
```
3.在所有机器上执行关闭节点命令

```
for i in `ps -ef |grep 'Elasticsearch -d'  |grep -v grep |awk '{print $2}'`; do kill $i; done
```

3.检查是否关闭 

```
ps -ef |grep 'Elasticsearch -d'
```

4.确认所有节点关闭后，依次重启所有节点

```
bin/elasticsearch -d
```

5.检测集群状态和节点 

```
_cat/health; _cat/nodes
```

6.等待集群恢复，等所有节点加入到集群并恢复yellow状态后才进行下一步操作

7. 所有节点加入到集群并恢复yellow状态后开启分片移动
```
curl  -XPUT 192.168.199.101:9206/_cluster/settings -H 'Content-Type: application/json' -d'{"persistent":{"cluster.routing.allocation.enable":null}}'
```

8.继续观察集群状态和节点

```
_cat/health; _cat/nodes
```



### 主机下线

遇到某台ES主机需要下线时，需要先排除该主机上的数据，待数据移走后，即可停节点下线主机。
根据主机ip排除 cluster.routing.allocation.exclude._ip
根据节点名称排除 cluster.routing.allocation.exclude._name

```
#排除节点
PUT _cluster/settings
{
  "transient":{
  "cluster.routing.allocation.exclude._name":"es7.2-192.168.199.101_9207"
  }
}
#停止排除节点
PUT _cluster/settings
{
  "transient":{
  "cluster.routing.allocation.exclude._name":null
  }
}
```

### 推迟分片移动

 Elasticsearch 将自动在可用节点间进行分片均衡，包括新节点的加入和现有节点的离线。理论上来说，这个是理想的行为，我们想要提拔副本分片来尽快恢复丢失的主分片。 我们同时也希望保证资源在整个集群的均衡，用以避免热点。

默认情况，集群会等待1分钟来查看节点是否会重新加入，如果这个节点在此期间重新加入，重新加入的节点会保持其现有的分片数据，不会触发新的分片分配。

在实践中，立即的再均衡所造成的问题会比其解决的更多，如果是主机临时故障，该主机上的节点分片会在其它节点重新生成，在数据量大的时候对网络，磁盘开销很大，而且在该主机故障恢复时，该主机上的节点再次加入集群时，该节点当前的数据已经没有用了

为了解决这种瞬时中断的问题，Elasticsearch 可以推迟分片的分配。这可以让你的集群在重新分配之前有时间去检测这个节点是否会再次重新加入。

```
#推迟分片移动 默认1m,修改默认延时
PUT /_all/_settings 
{
  "settings": {
    "index.unassigned.node_left.delayed_timeout": "20m" 
  }
}
```

https://www.elastic.co/guide/cn/elasticsearch/guide/current/_delaying_shard_allocation.html





### 磁盘容量不足

当磁盘快满时，所有索引会进入 read-only 状态, 出现数据无法继续写入情况

cluster.routing.allocation.disk.watermark.low: 低水位，默认 85% 当达到时，副本replica 不再写入 
cluster.routing.allocation.disk.watermark.high: 高水位，默认 90% 当达到时，分片shards 尝试写入到其他节点

 cluster.routing.allocation.disk.watermark.flood_stage: 默认 95% 当达到时，所有索引变为 readonly状态

- 解决办法：提升磁盘水位 或 手动清理数据

```json
PUT /_cluster/settings
{
  "transient": {
    "cluster.routing.allocation.disk.watermark.low":"90%",
    "cluster.routing.allocation.disk.watermark.high":"95%"
  }
}
```

 https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-cluster.html#disk-based-shard-allocation



### 数据导出

1.下载elastic-dump插件  nodejs项目，已做成离线安装包（elasticdump.zip）

​    https://github.com/elasticsearch-dump/elasticsearch-dump

2.解压命令 unzip elasticdump.zip

3.导出命令

 elastic-dump/node_modules/elasticdump/bin/elasticdump --input=http://elastic:elastic@localhost:9206/megacorp  --output=/home/ivory/es7.2.0download/megacorp.json --type=data  --limit 5000

4.导入到新索引 

elastic-dump/node_modules/elasticdump/bin/elasticdump --input=//home/ivory/es7.2.0download/megacorp.json --output=http://elastic:elastic@localhost:9206/megacorp-dump --type=data



## 安全认证

1.生成证书

```
./bin/elasticsearch-certgen
```

要求输入实例名称：elasticsearch-7.2.0

生成两个文件夹，一个ca, 一个实例名相同elasticsearch-7.2.0的文件夹

#### 2.证书放config目录下

证书ca文件夹和elasticsearch-7.2.0文件夹放config目录下

```
xpack.security.transport.ssl.verification_mode: certificate
xpack.security.enabled: true
xpack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.key: elasticsearch-7.2.0/elasticsearch-7.2.0.key
xpack.security.transport.ssl.certificate: elasticsearch-7.2.0/elasticsearch-7.2.0.crt
xpack.security.transport.ssl.certificate_authorities: ca/ca.crt
```

#### 3.重启所有es节点

#### 4.生成elastic及其它用户密码

生成elastic及其它用户

```
./bin/elasticsearch-setup-passwords interactive
```



增加权限后需要用户密码才能访问

curl  192.168.199.101:9202 不能访问，提示异常

curl -u elastic:elastic 192.168.199.101:9206

**kibana增加访问用户密码配置**

vi config/kibana.yml

elasticsearch.username: "elastic"
elasticsearch.password: "elastic"

**重启kibana**

 netstat -tunlp|grep 5601 

kill xx

nohup bin/kibana &



## 权限控制

1.先添加角色，并赋予角色权限

创建shsnc角色，赋予shsnc*索引all所有权限，集群层面monitor权限

2.创建用户，赋予用户角色

创建shsnc用户，赋予shsnc角色

创建索引

curl  -u shsnc:elastic -XPUT localhost:9206/customer   //提示未授权

curl  -u shsnc:elastic -XPUT localhost:9206/shsnc-amp //创建成功

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/security-privileges.html#privileges-list-indices
