# Hadoop 集群安装和配置

## 一、安装

### 1、用户设置

```
# 创建用户组
groupadd -g 102 hadoopgroup
# 创建用户
useradd -d /opt/hadoop -u 10201 -g 102 hadoop
# 设置用户密码
passwd hadoop
```

### 2、hosts设置

```
192.168.10.157  hadoop1
192.168.10.158  hadoop2
192.168.10.160  hadoop3
```

### 3、免密设置

```
# 切换用户
su hadoop
# 创建SSH秘钥，一路回车下去
ssh-keygen -t rsa
# 发送到目标机器
ssh-copy-id hostname
```

### 4、ftp工具安装

```
# 安装
yum -y install vsftpd
# 启动
systemctl start vsftpd.service
# 停止
systemctl stop vsftpd.service
# 重启
systemctl restart vsftpd.service
```

### 5、Java安装

[详见](./java-install)

### 6、Hadoop安装

```
tar -zxvf hadoop-3.1.1.tar.gz
mv hadoop-3.1.1 /opt/hadoop
```

### 7、环境变量

```
export JAVA_HOME=/usr/java/jdk1.8.0_181-amd64
export CLASSPATH=.:$JAVA_HOME/lib:$JAVA_HOME/jre/lib
export HADOOP_HOME=/opt/hadoop/hadoop-3.1.1
export PATH=$HADOOP_HOME/bin:$HADOOP_HOME/sbin:JAVA_HOME/bin:$PATH
```

## 二、配置

### 1、hadoop-env.sh

```
export JAVA_HOME=/usr/java/jdk1.8.0_181-amd64
export HDFS_DATANODE_SECURE_USER=root
export HDFS_NAMENODE_USER=root
export HDFS_DATANODE_USER=root
export HDFS_SECONDARYNAMENODE_USER=root
export YARN_NODEMANAGER_USER=root
export YARN_RESOURCEMANAGER_USER=root
```

### 2、core-site.xml

```
<configuration>
  <property>
    <name>hadoop.tmp.dir</name>
    <value>/opt/hadoop/hadoop-3.1.1/tmp</value>
  </property>
  <property>
    <name>fs.defaultFS</name>
    <value>hdfs://{hostname}:8020</value>
  </property>
</configuration>
```

### 3、yarn-site.xml

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

### 4、mapred-site.xml

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

### 5、hdfs-site.xml

```
<configuration>
  <property>
    <name>dfs.namenode.http-address</name>
    <value>hadoop1:50070</value>
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

### 6、格式化

```
hdfs namenode -format
```

### 7、启动集群

```
cd $HADOOP_HOME/sbin
./start-all.sh
```



