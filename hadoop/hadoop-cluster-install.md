# Hadoop 安装和配置

## 一、安装

### 1、下载

去Apache官方下载。

### 2、解压安装

```
tar -zxvf hadoop-3.1.1.tar.gz
mv hadoop-3.1.1 /opt/hadoop
```

### 3、环境变量

```
export HADOOP_HOME=/opt/hadoop/hadoop-3.1.1
export HADOOP_PREFIX=$HADOOP_HOME
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export HADOOP_INSTALL=$HADOOP_HOME
export PATH=$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$PATH
```

## 二、配置

### 1、创建目录

```
su hadoop
mkdir -p /data/hadoop/name
mkdir -p /data/hadoop/data
mkdir -p /data/hadoop/log
mkdir -p /data/hadoop/tmp
```

### 2、hadoop-env.sh

```
export JAVA_HOME=/usr/java/jdk1.8.0_181-amd64

export HADOOP_NAMENODE_OPTS=" -Xms1024m -Xmx1024m -XX:UseParallelGC"
export HADOOP_DATANODE_OPTS=" -Xms1024m -Xmx1024m"
export HADOOP_LOG_DIR=/data/hadoop/logs
```

### 3、core-site.xml

```
<configuration>
  <property>
    <name>hadoop.tmp.dir</name>
    <value>/data/hadoop/tmp</value>
  </property>
  <property>
    <name>fs.defaultFS</name>
    <value>hdfs://{hostname}:8020</value>
  </property>
</configuration>
```

### 4、yarn-site.xml

```
<configuration>
  <property>
    <name>yarn.resourcemanager.hostname</name>
    <value>hadoop1</value>
  </property>
  <property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
  </property>
  <property>
    <!-- 关闭内存检测，虚拟机需要，不配会报错-->
    <name>yarn.nodemanager.veme-check-enabled</name>
    <value>false</value>
  </property>
</configuration>
```

### 5、mapred-site.xml

```
<configuration>
  <property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
    <!-- local:本地运行，classic:经典mapreduce框架，yarn:新的框架 -->
  </property>
  <property>
    <name>mapreduce.admin.user.env</name>
    <value>HADOOP_MAPRED_HOME=/opt/hadoop/hadoop-3.1.1</value>
  </property>
  <property>
    <name>yarn.app.mapreduce.am.env</name>
    <value>HADOOP_MAPRED_HOME=/opt/hadoop/hadoop-3.1.1</value>
  </property>
</configuration>
```

### 6、hdfs-site.xml

```
<configuration>
  <property>
    <name>dfs.namenode.http-address</name>
    <value>hadoop1:50070</value>
  </property>
  <property>
    <name>dfs.namenode.name.dir</name>
    <value>/data/hadoop/name</value>
  </property>
  <property>
    <name>dfs.datanode.data.dir</name>
    <value>/data/hadoop/data</value>
  </property>
  <property>
    <name>dfs.replication</name>
    <value>3</value>
  </property>
  <property>
    <name>dfs.permissions.enabled</name>
    <value>false</value>
  </property>
  <property>
    <name>dfs.blocksize</name>
    <value>134217728</value>
  </property>
</configuration>
```

### 7、workers

```
# vim workers
hadoop1
hadoop2
hadoop3

# 快速配置
echo 'hadoop1' >> workers
echo 'hadoop2' >> workers
echo 'hadoop3' >> workers
```

### 8、格式化

```
hdfs namenode -format
```

## 三、管理

```
cd $HADOOP_HOME/sbin
```

### 1、启动&停止集群

```
# 启动
./start-all.sh
# 停止
./stop-all.sh
```

```
hadoop1# jps
64320 Jps
62577 SecondaryNameNode
62261 NameNode
62841 ResourceManager
62954 NodeManager
62380 DataNode

hadoop2# jps
59636 Jps
59466 NodeManager
59036 SecondaryNameNode
58845 DataNode

hadoop3# jps
59093 Jps
58641 NodeManager
58494 DataNode
```

### Web控制台

```
# Hadoop 集群管理
http://192.168.10.110:8088
# DFS 管理
http://192.168.10.110:50070
```

## 其它

使用`jps`查看进程

### 1、NameNode

### 2、DataNode

### 3、SecondaryNameNode

### 4、TaskTracker

### 5、JobTracker

