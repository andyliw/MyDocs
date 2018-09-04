# Zookeeper 集群安装和管理

## 一、安装

### 1、解压安装

```
tar -zxvf zookeeper-3.4.13.tar.gz
mv zookeeper-3.4.13 /opt/hadoop
```

### 2、环境变量

```
# su hadoop
# vim ~/.bashrc

# zookeeper
export ZOOKEEPER_HOME=/opt/hadoop/zookeeper-3.4.13
PATH=$ZOOKEEPER_HOME/bin:$PATH
```

### 3、数据目录

```
mkdir -p /data/zookeeper/data
mkdir -p /data/zookeeper/logs
```

### 4、myid

```
touch /data/zookeeper/data/myid
echo '{id}' > /data/zookeeper/data/myid
# ID 就是123按序写入
```

### 5、zookeeper-env.sh

```
#!/usr/bin/env bash

export ZOO_LOG_DIR=/data/zookeeper/logs
export ZOO_LOG4J_PROP=INFO,ROLLINGFILE
```

### 6、zoo.cfg

```
# $ZOOKEEPER_HOME/conf/zoo.cfg

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

## 二、管理

### 1、启停管理

```
cd $ZOOKEEPER_HOME/bin
# 启动
./zkServer.sh start
# 查看状态
./zkServer.sh status
# 停止
./zhServer.sh stop
```

### 2、注册服务

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

### 3、客户端

```
cd $ZOOKEEPER_HOME/bin

./zkCli.sh -server hadoop1:2181
```
