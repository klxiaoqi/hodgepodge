# es添加删除节点

# 添加节点
> **步骤:**  
> 1. 拷贝原有节点的ES相关文件到新机器 
> 2. 修改核心配置文件jvm.options和elasticsearch.yml
> 3. 访问9200端口验证成功与否。

> **注意**： 
> 1. ES必须版本号和其他Elasticsearch节点一致 
> 2. jvm注意结合实际机器的内存进行合理化配置。取值：Min（32GB，机器内存一半）
> 3. 根据分配的角色（Master/data/client）配置 
> 4. 集群名称必须和预先的机器一致 
> 5. 避免脑裂，合理化如下配置
```
curl -XPUT 'localhost:9200/_cluster/settings' -d'
{
  "transient": {
    "discovery.zen.minimum_master_nodes": 3
  }
}
```

---

# 删除节点

## 方案1--排除停用节点

> 1. 告知群集排除停用节点

```
PUT _cluster/settings
{
  "transient" : {
    "cluster.routing.allocation.exclude._ip" : "10.0.0.1"
  }
}
```

> *这将导致Elasticsearch将该节点上的分片分配给其余节点，而不会将群集状态更改为黄色或红色（即使您的副本数设置为0）。重新分配所有分片后，您可以关闭节点并执行您需要执行的任何操作。 完成后，Elasticsearch将再剩余节点上再次重新平衡分片。*

> 2. 检查集群健康状态
```
curl -XGET http://ES_SERVER:9200/_cluster/health?pretty
```
> *如果没有节点relocating，则排除节点已经被安全剔除，可以考虑关闭节点*

> 3. 判定数据是否还存在
```
curl -XGET http://ES_SERVER:9200/_nodes/NODE_NAME/stats/indices?pretty
```