
- 下载解压缩
  ``` bash
    wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.6.0.tar.gz
    tar -zxvf elasticsearch-5.6.0.tar.gz
  ```
- 配置
  ``` bash
  cd elasticsearch-5.6.0/config
  vi elasticsearch.yml
  # 修改
  cluster.name: es-test
  node.name: node-n4 # node-n5,node-n6
  network.host: 0.0.0.0
  discovery.zen.ping.unicast.hosts:["test-n4","test-n5", "test-n6"]

  ```
  > 设置系统参数 vm.max_map_count
  ``` bash
  vi /etc/sysctl.conf 
    vm.max_map_count=262144
  sudo sysctl –p
  ```
- 启动
  ```
  bin\elasticsearth
  ```
> 通过web http://10.8.30.35:9200/_cat/health?v&pretty 查看集群状态

- 创建索引
  
  ``` bash
  # 创建savoir_themes索引以及映射. 分片数3，副本数1
  PUT /savoir_themes  {
    "number_of_shards": 3,
    "number_of_replicas": 1,
    "settings": {
    "index.mapping.single_type" : true
    },
    "mappings": {
      "theme" : {
        "properties": {
          "sensor": {"type": "integer"},
          "factor": {"type": "integer"},
		      "iota_device": {"type": "keyword"},
          "collect_time": {
            "type": "date",
            "format": "date_time || yyy-MM-dd HH:mm:ss"
          },
          "data" : {
            "properties": {
              "deflection" : {"type" : "double"},
              "max" : {"type" : "double"},
              "min" : {"type" : "double"},
              "soundLevel" : {"type" : "double"},
              "ppv" : {"type" : "double"},
              "pv" : {"type" : "double"},
              "trms" : {"type" : "double"},
              "temperature" : {"type" : "double"},
              "humility" : {"type" : "double"}
            }
          }
        }
      }
    }
  }

  ```