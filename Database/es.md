

## es

```
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.1.0-darwin-x86_64.tar.gz
```



```
查看插件
bin/elasticsearch-plugin list
安装插件
bin/elasticsearch-plugin install analysis-icu
接口方式查看插件
curl http://127.0.0.1:9200/_cat/plugins
```



```bash
#es  单机多实例
bin/elasticsearch -E node.name=node0 -E http.port=9200 -E cluster.name=cluster1 -E path.data=node0_data -d
bin/elasticsearch -E node.name=node1 -E http.port=9201 -E cluster.name=cluster1 -E path.data=node1_data -d
bin/elasticsearch -E node.name=node2 -E http.port=9202 -E cluster.name=cluster1 -E path.data=node2_data -d
bin/elasticsearch -E node.name=node3 -E http.port=9203 -E cluster.name=cluster1 -E path.data=node3_data -d
```



```
curl http://127.0.0.1:9200/_cat/nodes
```





## kibana

```
wget https://artifacts.elastic.co/downloads/kibana/kibana-7.1.0-darwin-x86_64.tar.gz
```



```
./bin/kibana
```



http://127.0.0.1:5601/



查看插件

```
bin/kibana-plugin list
```



dev_tools



```
# 执行es 的api 
GET _cat/nodes
```





cerebro

## logstash



```
wget  https://artifacts.elastic.co/downloads/logstash/logstash-7.1.0.tar.gz
```



test data

```
wget https://files.grouplens.org/datasets/movielens/ml-latest-small.zip
```



logstash-movielens.conf

```
input {
  file {
    path => "/Users/yandun/es/logstash-test-data/ml-latest-small/movies.csv"
    start_position => "beginning"
    sincedb_path => "/dev/null"
  }
}
filter {
  csv {
    separator => ","
    columns => ["id","content","genre"]
  }

  mutate {
    split => { "genre" => "|" }
    remove_field => ["path", "host","@timestamp","message"]
  }

  mutate {

    split => ["content", "("]
    add_field => { "title" => "%{[content][0]}"}
    add_field => { "year" => "%{[content][1]}"}
  }

  mutate {
    convert => {
      "year" => "integer"
    }
    strip => ["title"]
    remove_field => ["path", "host","@timestamp","message","content"]
  }

}
output {
   elasticsearch {
     hosts => "http://localhost:9200"
     index => "movies"
     document_id => "%{id}"
   }
  stdout {}
}
```





```
./bin/logstash -f logstash-movielens.conf 
```



```
GET _cat/indices/movies
```





## es  conception



```
索引 文档 

节点 分片

文档 --- 关系型数据库中的一条记录  文档会被序列化为json格式保存在es中
文档中的元数据
_index  文档所属索引名字
_type 文档所属类型名字
_source 文档所有原始信息
_version 文档版本 可以解决写并发冲突问题
_score  相关性打分

索引 一类相似文档的集合  每一个索引都有一个自己的mapping 定义文件 用于描述文档的字段名 和 字段类型


shard 分片  体现的是物理空间的概念  索引中的数据最后被分散在shard分片上

可以为索引设置 mapping 和 setting
mapping 定义文档中字段的类型
setting 定义不同的数据分布  用到分片


type  每个索引只能创建一个type



es 和 关系型数据库 使用场景
进行全文检索 对搜索结果进行算分  适合es
对数据的事务性要求比较高 适合关系型数据库  或者结合使用二者
```





```
GET  movies 
GET  movies/_count // 索引文档数量
```









## es cluster



```
#节点
Master-eligible nodes 和 Master eligible节点
有资格成为master 节点的节点  可以修改集群状态信息，如果允许每个节点都可以修改集群信息，会导致节点信息的混乱

data 节点 负责保存分片数据

Coordinating Node
负责接受Client的请求,将请求分发到合适的节点,最终把结果汇聚到一起
每个节点默认起到了Coordinating Node的职责
  
  
  
Hot & Warm Node 
不同硬件配置的Data Node,用来实现Hot&Warm架构,降低集群部署成本


Machine Learning Node
负责跑机器学习的Job,用来坐异常检测


#分片 有两类
主分片   解决数据水平扩张问题  通过主分片可以把一个索引上的数据分散到集群内所有节点上去。一个主分片就是一个运行的lucene实例
主分片数在创建索引时指定 不允许修改 如果修改需要reindex 进行


分片副本  解决数据高可用的问题  保证数据不会产生真正的丢失 每个索引的副本分片数量时可以动态调整的
通增加副本分片的数据 可以在一定程度上提高数据查询的性能


```



