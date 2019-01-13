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
  export HADOOP_HOME=/home/anxin/hadoop-2.7.4
  export PATH=$PATH:$HADOOP_HOME/bin
  ``` 
  
  - 修改hadoop配置文件
  
  ```
  # 修改./etc/hadoop/hadoop-env.sh
  
  
  ```