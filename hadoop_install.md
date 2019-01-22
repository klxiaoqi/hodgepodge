- 下载解压
  ```
  wget  http://mirrors.hust.edu.cn/apache/hadoop/common/hadoop-2.7.4/hadoop-2.7.4.tar.gz
  tar -zxvf hadoop-2.7.4.tar.gz
  ```
- 配置
  ``` bash
  mv hadoop-2.7.4 hadoop
  cd hadoop
  mkdir -p tmp hdfs/data hdfs/name
  ```  

  - 增加环境变量
  
  ```
  # 编辑 /etc/profile
  export HADOOP_HOME=/home/test/hadoop
  export PATH=$PATH:$HADOOP_HOME/bin
  ``` 
  
  - 修改hadoop配置文件
  
  > 修改 hadoop/etc/hadoop/hadoop-env.sh
  
  ``` bash
  export JAVA_HOME=/usr/lib/jdk1.8.0_131
  export HADOOP_OPTS="$HADOOP_POTS -Djava.library.path=${HADOOP_HOME}/lib/native"
  ```
  
  > 修改 hadoop/etc/hadoop/core-site.xml
  
  ``` xml
  <configuration>
    <property>
      <name>fs.defaultFS</name>
      <value>hdfs://test-n4:9000</value>
    </property>
    <property>
      <name>hadoop.tmp.dir</name>
      <value>file:/home/test/hadoop/tmp</value>
    </property>
    <property>
      <name>fs.hdfs.impl.disable.cache</name>
      <value>true</value>
    </property>
  </configuration>

  ```
  > 修改 hadoop/etc/hadoop/hdfs-site.xml
  
  ``` xml
  <configuration>
    <property>
      <name>dfs.replication</name>
      <value>1</value>
    </property>
    <property>
      <name>dfs.namenode.name.dir</name>
      <value>file:/home/test/hadoop/hdfs/name</value>
    </property>
    <property>
      <name>dfs.datanode.data.dir</name>
      <value>file:/home/test/hadoop/hdfs/data</value>
    </property>
    <propery>
      <name>dfs.support.append</name>
      <value>true</value>>
    </property>
  </configuration>

  ```

  > 修改 hadoop/etc/hadoop/mapred-site.xml
  
  ``` xml
  <configuration>
    <property>
      <name>mapred.job.tracker</name>
      <value>test-n4:9001</value>
    </property>
  </configuration>

  ```
  
  >修改 hadoop/etc/hadoop/yarn-site.xml
  
  ``` xml
  <configuration>
    <!-- Site specific YARN configuration properties -->
    <property>
      <name>yarn.nodemanager.resource.memory-mb</name>
      <value>2048</value>
      <description>后续spark任务需要</description>
    </property>
    <property>
      <name>yarn.nodemanager.resource.cpu-vcores</name>
      <value>1</value>
      <description>此项配置可不配</description>
    </property>
    <property>
      <name>yarn.nodemanager.aux-services</name>
      <value>mapreduce_shuffle</value>
    </property>
    <property>
      <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
      <value>org.apache.hadoop.mapred.ShuffleHandler</value>
    </property>
    <property>
      <name>yarn.resourcemanager.address</name>
      <value>test-n4:8032</value>
    </property>
    <property>
      <name>yarn.resourcemanager.scheduler.address</name>
      <value>test-n4:8030</value>
    </property>
    <property>
      <name>yarn.resourcemanager.resource-tracker.address</name>
      <value>test-n4:8031</value>
    </property>
    <property>
      <name>yarn.resourcemanager.admin.address</name>
      <value>test-n4:8033</value>
    </property>
    <property>
      <name>yarn.resourcemanager.webapp.address</name>
      <value>test-n4:8088</value>
    </property>
  </configuration>

  ```

  > 修改 hadoop/etc/hadoop/slaves (workers)
  
  ```
  test-n4
  test-n5
  test-n6
  ```
  
- 启动

  - 格式化hdfs文件系统

  ``` bash
  hadoop namenode –format
  ```

  - 启动 hdfs
  
  ``` bash
  sbin/start-dfs.sh
  ```  
  > 测试
  
  ``` bash
  hadoop dfs -mkdir /test
  hdfs dfs -ls /
  ```
  
  - 启动 yarn
  
  ``` bash
  sbin/start-yarn.sh
  # 查看进程
  jsp
  ```

 > 可以通过打开 test-n4:50070 观察hdfs状态  
  