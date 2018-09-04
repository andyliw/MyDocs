# HBase 安装

## 一、安装

### 环境变量

```
# vim ~/.bashrc

export HBASE_HOME=/opt/hadoop/hbase-2.1.0
export PATH=$HBASE_HOME/bin:$PATH
```

### 数据目录

```
mkdir -p /data/hbase/data
mkdir -p /data/hbase/logs
mkdir -p /data/hbase/tmp
```

### hbase-env.sh

```
export JAVA_HOME=/usr/java/jdk1.8.0_181-amd64
export HBASE_CLASSPATH=/opt/hadoop/hadoop-3.1.1/etc/hadoop
export HBASE_MANAGES_ZK=false
export HBASE_LOG_DIR=/data/hbase/logs
```

### hbase-site.xml

```
<configuration>
  <property>
    <name>hbase.rootdir</name>
    <value>hdfs://ygxt/hbase</value>
  </property>
  <property>
    <name>hbase.cluster.distributed</name>
    <value>true</value>
  </property>
  <property>
    <name>hbase.zookeeper.quorum</name>
    <value>hadoop3:2181,hadoop4:2181,hadoop5:2181</value>
  </property>
</configuration>
```

### 管理

```
# 启动
start-hbase.sh
# 停止
stop-hbase.sh
```

### 测试

```
# ./hbase shell

> create 'testTable','testFamily'
> pub 'testTable','row1','testFamily:name','jack'
> scan 'testTable'
```