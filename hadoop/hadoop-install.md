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
export HDFS_DATANODE_SECURE_USER=root
export HDFS_NAMENODE_USER=root
export HDFS_DATANODE_USER=root
export HDFS_SECONDARYNAMENODE_USER=root
export YARN_NODEMANAGER_USER=root
export YARN_RESOURCEMANAGER_USER=root
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

### 7、格式化

```
hdfs namenode -format
```

### 8、启动集群

```
cd $HADOOP_HOME/sbin
./start-all.sh
```

## 三、管理

使用`jps`查看进程

### 1、NameNode

### 2、DataNode

### 3、SecondaryNameNode

### 4、TaskTracker

### 5、JobTracker

