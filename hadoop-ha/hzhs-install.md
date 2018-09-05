# Zookeeper+Hadoop+HBase+Spark HA安装

## 一、版本

Hadoop 不能选择最新版本（有坑），2.7的版本比较稳定，而且HBase的最新版本(2.1.0)依赖的Hadoop版本为2.7.4。
Zookeeper、HBase、Spark版本可选较新的稳定版。

```
zookeeper: 3.4.13
hadoop: 2.7.7
hbase: 2.0.2
spark: 2.3.1
java: 1.8.X
scala: 2.11.X
```

## 二、规划

### 1、机器

测试用，计划5台机器。

### 2、进程

|Hostname |Process  |
|:--------|:--------|
|hadoop1  |NameNode(hdfs), DFSZKFailoverController(hdfs), ResourceManager(yarn), HMaster(hbase), Master(spark)|
|hadoop2  |NameNode(hdfs), DFSZKFailoverController(hdfs), ResourceManager(yarn), HMaster(hbase), Master(spark)|
|hadoop3  |QuorumPeerMain(zookeeper), JournamNode(hdfs), DataNode(hdfs), NodeManager(yarn), HRegionServer(hbase), Worker(spark)|
|hadoop4  |QuorumPeerMain(zookeeper), JournamNode(hdfs), DataNode(hdfs), NodeManager(yarn), HRegionServer(hbase), Worker(spark)|
|hadoop5  |QuorumPeerMain(zookeeper), JournamNode(hdfs), DataNode(hdfs), NodeManager(yarn), HRegionServer(hbase), Worker(spark)|

### 3、启动序

```
zookeeper:
  QuorumPeerMain
hdfs:
  JournamNode
  NameNode
  DFSZKFailoverController
  DataNode
yarn:
  ResourceManager
  NodeManager
hbase:
  HMaster
  HRegionServer
spark:
  Master
  Worker
```

## 三、系统

### 1、安装CentOS7

安装CentOS7。

### 2、hosts设置

```
192.168.10.110  hadoop1
192.168.10.113  hadoop2
192.168.10.112  hadoop3
192.168.10.114  hadoop4
192.168.10.115  hadoop5
```

### 3、用户设置

```
# 创建用户组
groupadd -g 102 hadoop
# 创建用户
useradd -d /opt/hadoop -u 10201 -g 102 hadoop
# 设置用户密码
passwd hadoop
```

### 4、创建目录

```
mkdir -p /data
mkdir -p /opt/hadoop
chown -R hadoop:hadoop /data
chown -R hadoop:hadoop /opt/hadoop
```

### 5、sudo

```
# vim /etc/sudoers

hadoop  ALL=NOPASSWD:ALL
```

### 6、免密设置

sshd_config设置：

```
# 禁止空密码用户登录
PermitEmptyPasswords no
# 开启密码登录授权(默认开启)
PasswordAuthentication yes
# 禁止root账户使用ssh登录，防止提权后用root账户破坏
PermitRootLogin no
```

authorized_keys设置：

```
# 切换用户
su hadoop
# 创建SSH秘钥，一路回车下去
ssh-keygen -t rsa
# 发送到目标机器
ssh-copy-id hostname
```

## 四、安装

### 1、Java安装

[详见](../centos7/java-install)

### 2、Scala安装

[详见](../centos7/scala-install)

### 3、Zookeeper

解压安装：

```
tar -zxvf zookeeper-3.4.13.tar.gz -C /opt/hadoop
```

环境变量：

```
# vim ~/.bashrc

# Zookeeper
export ZOOKEEPER_HOME=/opt/hadoop/zookeeper-3.4.13
export PATH=$PATH:$ZOOKEEPER_HOME/bin
```

### 4、Hadoop

解压安装：

```
tar -zxvf hadoop-2.7.7.tar.gz -C /opt/hadoop
```

环境变量：

```
# vim ~/.bashrc

# Hadoop
export HADOOP_HOME=/opt/hadoop/hadoop-2.7.7
export HADOOP_PREFIX=$HADOOP_HOME
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export HADOOP_INSTALL=$HADOOP_HOME
export PATH=$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$PATH
```

### 5、HBase

解压安装：

```
tar -zxvf hbase-2.0.2-bin.tar.gz -C /opt/hadoop
```

环境变量：

```
# vim ~/.bashrc

# HBase
export HBASE_HOME=/opt/hadoop/hbase-2.0.2
export PATH=$PATH:$HBASE_HOME/bin
```

### 6、Spark

解压安装：

```
tar -zxvf spark-2.3.1-bin-hadoop2.7.tgz -C /opt/hadoop
```

环境变量：

```
# vim ~/.bashrc

# Spark
export SPARK_HOME=/opt/hadoop/spark-2.3.1
export PATH=$PATH:$SPARK_HOME/bin
```

## 五、Zookeeper配置启动

### 1、创建目录

```
mkdir -p /data/zookeeper/data
mkdir -p /data/zookeeper/logs
```

### 2、修改配置

```
cd $ZOOKEEPER_HOME/conf
```

zookeeper-env.sh

```
#!/usr/bin/env bash

export ZOO_LOG_DIR=/data/zookeeper/logs
export ZOO_LOG4J_PROP=INFO,ROLLINGFILE
```

zoo.cfg

```
tickTime=2000
initLimit=5
syncLimit=2
dataDir=/data/zookeeper/data
dataLogDir=/data/zookeeper/logs
clientPort=2181
server.1=hadoop3:2888:3888
server.2=hadoop4:2888:3888
server.3=hadoop5:2888:3888
maxClientCnxns=60
minSessionTimeout=4000
maxSessionTimeout=300000

# 注释：
# tickTime：心跳时间
# initLimit：多少个心跳时间内，允许其他server连接并初始化数据
# syncLimit：多少个tickTime内，允许follower节点同步
# dataDir：存放内存数据文件目录，根据实际环境修改
# dataLogDir：存放日志文件目录，根据实际环境修改
# clientPort：监听端口，使用默认2181端口
# server.x:配置集群主机信息，[hostname]:[通信端口]:[选举端口]，根据自己的主机信息修改
# maxClientCnxns：最大并发客户端数，用于防止DOS的，设置为0是不加限制
# minSessionTimeout：最小的客户端session超时时间（单位是毫秒）
# maxSessionTimeout：最大的客户端session超时时间（单位是毫秒）
```

### 3、集群序号

```
# id 为数值，每台机器不能重复，按序写入较好理解
echo {id} > /data/zookeeper/data/myid
```

### 4、启停管理

```
# cd $ZOOKEEPER_HOME/bin

# 启动
zkServer.sh start
# 查看状态
zkServer.sh status
# 停止
zhServer.sh stop
# 命令行
zkCli.sh -server hadoop1:2181
```

### 5、注册服务

```
# vim /usr/lib/systemd/system/zookeeper.service

[Unit]
Description=Zookeeper Service
After=syslog.target network.target

[Service]
Type=forking
User=hadoop
Group=hadoop
SyslogIdentifier=hadoop
ExecStart=/opt/hadoop/zookeeper-3.4.13/bin/zkServer.sh start
ExecStop=/opt/hadoop/zookeeper-3.4.13/bin/zkServer.sh stop
Restart=always

[Install]
WantedBy=multi-user.target

# 注册服务&启动服务
# systemctl enable zookeeper.service
# systemctl start zookeeper.service
```

## 六、Hadoop配置启动

### 1、创建目录

```
su hadoop
mkdir -p /data/hadoop/data
mkdir -p /data/hadoop/name
mkdir -p /data/hadoop/journal
mkdir -p /data/hadoop/logs
mkdir -p /data/hadoop/tmp
```

### 2、修改配置

```
cd $HADOOP_HOME/etc/hadoop
```

hadoop-env.sh

```
export JAVA_HOME=/usr/java/jdk1.8.0_181-amd64
export HADOOP_LOG_DIR=/data/hadoop/logs
# export HADOOP_NAMENODE_OPTS=" -Xms1024m -Xmx1024m"
# export HADOOP_DATANODE_OPTS=" -Xms1024m -Xmx1024m"
```

core-site.xml

```
<configuration>
  <property>
    <name>fs.defaultFS</name>
    <value>hdfs://ygxt</value>
  </property>
  <property>
    <name>hadoop.tmp.dir</name>
    <value>/data/hadoop/tmp</value>
  </property>
  <property>
    <name>ha.zookeeper.quorum</name>
    <value>hadoop3:2181,hadoop4:2181,hadoop5:2181</value>
  </property>
  <property>
    <name>ha.zookeeper.session-timeout.ms</name>
    <value>30000</value>
    <description>测试环境提高超时</description>
  </property>
</configuration>
```

hdfs-site.xml

```
<configuration>
  <property>
    <name>dfs.replication</name>
    <value>1</value>
  </property>
  <property>
    <name>dfs.nameservices</name>
    <value>ygxt</value>
  </property>
  <property>
    <name>dfs.ha.namenodes.ygxt</name>
    <value>nn1,nn2</value>
  </property>
  <property>
    <name>dfs.ha.fencing.methods</name>
    <value>
      sshfence
      shell(/bin/true)
    </value>
  </property>
  <property>
    <name>dfs.ha.fencing.ssh.private-key-files</name>
    <value>/home/hadoop/.ssh/id_rsa</value>
  </property>
  <property>
    <name>dfs.ha.fencing.ssh.connect-timeout</name>
    <value>30000</value>
  </property>
  <property>
    <name>dfs.ha.automatic-failover.enabled</name>
    <value>true</value>
  </property>
  <property>
    <name>dfs.namenode.rpc-address.ygxt.nn1</name>
    <value>hadoop1:8020</value>
  </property>
  <property>
    <name>dfs.namenode.http-address.ygxt.nn1</name>
    <value>hadoop1:50070</value>
  </property>
  <property>
    <name>dfs.namenode.rpc-address.ygxt.nn2</name>
    <value>hadoop2:8020</value>
  </property>
  <property>
    <name>dfs.namenode.http-address.ygxt.nn2</name>
    <value>hadoop2:50070</value>
  </property>
  <property>
    <name>dfs.namenode.name.dir</name>
    <value>/data/hadoop/name</value>
  </property>
  <property>
    <name>dfs.namenode.shared.edits.dir</name>
    <value>qjournal://hadoop3:8485;hadoop4:8485;hadoop5:8485/ygxt</value>
  </property>
  <property>
    <name>dfs.journalnode.edits.dir</name>
    <value>/data/hadoop/journal</value>
  </property>
  <property>
    <name>dfs.qjournal.start-segment.timeout.ms</name>
    <value>60000</value>
  </property>
  <property>
    <name>dfs.client.failover.proxy.provider.ygxt</name>
    <value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
  </property>
  <property>
    <name>dfs.datanode.data.dir</name>
    <value>/data/hadoop/data</value>
  </property>
</configuration>
```

mapred-site.xml

```
<configuration>
  <property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
    <!-- local:本地运行，classic:经典mapreduce框架，yarn:新的框架 -->
  </property>
</configuration>
```

yarn-site.xml

```
<configuration>
  <property>
    <name>yarn.resourcemanager.ha.enabled</name>
    <value>true</value>
  </property>
  <property>
    <name>yarn.resourcemanager.cluster-id</name>
    <value>yrc</value>
  </property>
  <property>
    <name>yarn.resourcemanager.ha.rm-ids</name>
    <value>rm1,rm2</value>
  </property>
  <property>
    <name>yarn.resourcemanager.hostname.rm1</name>
    <value>hadoop1</value>
  </property>
  <property>
    <name>yarn.resourcemanager.hostname.rm2</name>
    <value>hadoop2</value>
  </property>
  <property>
    <name>yarn.resourcemanager.zk-address</name>
    <value>hadoop3:2181,hadoop4:2181,hadoop5:2181</value>
    <description>zookeeper cluster address.</description>
  </property>
  <property>
    <name>yarn.resourcemanager.recovery.enabled</name>
    <value>true</value>
  </property>
  <property>
    <name>yarn.resourcemanager.score.class</name>
    <value>org.apache.hadoop.yarn.server.resourcemanager.recovery.ZKRMStateStore</value>
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
  <property>
    <name>yarn.log-aggregation.enable</name>
    <value>true</value>
  </property>
  <property>
    <name>yarn.log-aggregation.retain-seconds</name>
    <value>108600</value>
  </property>
</configuration>
```

salves

```
# vim slaves
hadoop3
hadoop4
hadoop5

# 快速配置
rm -rf slaves
echo 'hadoop3' >> slaves
echo 'hadoop4' >> slaves
echo 'hadoop5' >> slaves
```

### 3、Hdfs初始化

* 1. JournamNode

启动 JournamNode，启动节点必须是奇数，按规划启动。

```
# hadoop3, hadoop4, hadoop5
hadoop-daemon.sh start journalnode
```

* 2. NameNode

在第一个NameNode节点(hadoop1)上执行格式化，格式化完成后初始化JournalNode节点，执行完成后启动它。

```
# 格式化 NameNode
hdfs namenode -format
# 初始化 JournalNode
hdfs namenode -initializeSharedEdits
# 启动 NameNode
hadoop-daemon.sh start namenode
```

在第二个NameNode节点(hadoop2)上执行清空指令，执行完成后启动它。

```
# 清空
hdfs namenode -bootstrapStandby
# 启动 NameNode
hadoop-daemon.sh start namenode
```

执行完成后查看管理页面。

```
http://hadoop1:50070
http://hadoop2:50070
```

* 3. DFSZKFailoverController(zkfc)

初始化Zookeeper集群。

```
hdfs zkfc -formatZK
```

在管理节点(hadoop1,hadoop2)启动zkfc

```
hadoop-daemon.sh start zkfc
```

* 4. DataNode

```
hadoop-daemon.sh start datanode
```

### 4、Yarn启停管理

```
# hadoop1
# 启动Yarn
start-yarn.sh

# hadoop2
# 启动RM
yarn-daemon.sh start resourcemanager

# 其它
# 关闭RM
yarn-daemon.sh stop resourcemanager
# 启动NM
yarn-daemon.sh start nodemanager
# 关闭NM
yarn-daemon.sh stop nodemanager
# 查看状态
yarn rmadmin -getServiceState <rm-id>
```



