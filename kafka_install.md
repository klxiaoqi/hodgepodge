
# zookeeper

> kafka 依赖zookeeper 管理自身集群(Broker, Offset, Producer, Consumer等),所以要先安装zookeeper.

 - 下载 
   
   ```
   wget http://mirror.bit.edu.cn/apache/zookeeper/zookeeper-3.4.10/zookeeper-3.4.10.tar.gz

   ```
 - 解压
    ```
        tar -zxvf zookeeper-3.4.10.tar.gz
    ```
 - 配置
   ``` bash
       cd zookeeper-3.4.10/conf
       cp zoo_sample.cfg zoo.cfg
       # 修改 dataDir(数据保存路径): 路径后面不能有注释
       dataDir=/home/test/zookeeper-3.4.10/data
       # 增加集群配置节点 hostname:port1:port2  port1 是node间的通信端口，
       # port2是选举端口
       server.0=test-n4:4001:4002
       server.1=test-n5:4001:4002
       server.2=test-n6:4001:4002
   ```
   > zookeeper 默认服务端口是：2181

   > 在上述配置的dataDir路径下创建一个 myid 文件

   ``` bash
    # 每个节点都增加 myid 内容是节点编号
    echo '0' >> myid
    # echo '1' >> myid
    # echo '2' >> myid
   ```
 - 运行
   ``` bash
   bin/zkServer.sh start
   # 查看状态
   bin/zkServer.sh status
   ```
   ```
    test@test-n3:~/services/zookeeper$ bin/zkServer.sh status
    ZooKeeper JMX enabled by default
    Using config: /home/test/services/zookeeper/bin/../conf/zoo.cfg
    Mode: leader

    test@test-n1:~/services/zookeeper$ bin/zkServer.sh status
    ZooKeeper JMX enabled by default
    Using config: /home/test/services/zookeeper/bin/../conf/zoo.cfg
    Mode: follower

    test@test-n3:~/services/zookeeper$ bin/zkServer.sh status
    ZooKeeper JMX enabled by default
    Using config: /home/test/services/zookeeper/bin/../conf/zoo.cfg
    Mode: leader

   ```
   > test-n3 是leader
   
 - 停止
    ```
    bin/zkServer.sh stop
    ```

# kafka

 - 下载并解压
    ```
    wget http://mirror.bit.edu.cn/apache/kafka/0.11.0.1/kafka_2.12-0.11.0.1.tgz
    tar -zxvf kafka_2.12-0.11.0.1.tgz
    ```
 - 配置
    ``` bash
    cd kafka_2.12-0.11.0.1
    vi config/server.properties
    # 修改
    broker.id=0 # 每个节点不能相同 1, 2
    delete.topic.enable=true
    listeners=PLAINTEXT://test-n1:9092
    log.dirs=/home/anxin/kafka_2.12-0.11.0.1/kafka-logs
    zookeeper.connect=test-n1:2181,test-n2:2181,test-n3:2181
    ```
 - 启动

    ```
    bin/kafka-server-start.sh -daemon config/server.properties

    ```
 - 查看topic状态
    ```
    bin/kafka-topics.sh --describe --zookeeper test-n1:2181,test-n2:2181,test-n3:2181 
--topic data

    ```
 - 常用命令
    ``` bash
    # 创建一个名为data的topic, 拥有1个分区，1个副本
    bin/kafka-topics.sh --create --zookeeper test-n1:2181,test-n2:2181,test-n3:2181 --replication-factor 1 --partitions 1 --topic data

    # 查看topic的状态

    bin/kafka-topics.sh --describe --zookeeper test-n1:2181,test-n2:2181,test-n3:2181 --topic data

    # 删除topic
    bin/kafka-topics.sh --delete --zookeeper test-n1:2181 --topic data

    # 查看所有topics
    

    ```    
