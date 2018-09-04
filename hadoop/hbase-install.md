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
mkdir -p /data/hbase/zookeeper
```

### hbase-site.xml

```
<configuration>
  <property>
    <name>hbase.rootdir</name>
    <value>/data/hbase/data</value>
  </property>
  <property>
    <name>hbase.zookeeper.property.dataDir</name>
    <value>/data/hbase/zookeeper</value>
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