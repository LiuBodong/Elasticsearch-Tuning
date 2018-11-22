# Elasticsearch文档（基于6.3.2）

## 一、调优（大数据量）

### 数据分布

+ 合理分配集群。客户端，数据节点，计算节点等。[详情](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-node.html)
+ 按照时间来分索引，适用于日志搜索等。比如可以划分为201801,201802。在删除过期时只需删除索引即可。
+ 如果不能按照时间来分，那么可以使用**routing**(类似于分区),具体做法是在建立索引的时候指定routing
    > 建立索引:client.prepareIndex("index1").setRouting("201801")...;  
    > 查询:client.prepareSearch("index1").setRouting("201801")...;  
    > 注意：查询时如果要忽略routing，不指定routing
+ 合理设置分片数，不宜过多或过少，要根据实际情况来选择。
+ 丢弃不需要的字段。
+ 建立索引时可以通过设定分片副本数**index.number_of_replicas=0**为0来加快速度。
+ 设置刷新时间，默认刷新时间为1s，会加大cpu负载，设置为60s。
    >Settings.builder().put("index.number_of_shards", 3)
    >.put("index.number_of_replicas", 0)
    >.put("index.refresh_interval", "60s")
    >.build()
+ 索引建立完毕，通过增大设置**index.number_of_replicas**来增加副本数，提高搜索性能。
  >Settings.builder().put("number_of_replicas", 1)
  >.put("refresh_interval", "60s")
  >.build()

### 设置

+ 使用特定的Mapping，不要使用自动生成的Mapping
+ 设置不用来排序，聚合的字段属性 **"doc_values=false"**。
+ 中文分词使用自定义词库和停词。
+ JVM内存最好不要超过32G，有个32G陷阱，可以设置为31G。
+ 内存尽量不要占用超过系统内存的一半，留给系统做缓存。
+ 增大查询缓存，默认为JVM内存的10%，可以适当提高到20%或更高。elasticsearch.yml中设置:
  > indices.queries.cache.size: 20%

### 编码&&架构设计

+ 尽量不用聚合，例如sum、avg、时间聚合等操作。
+ 搜索使用filter context代替query context
+ 对于翻页，使用**SearchAfter** API代替 from to API。

### 待续