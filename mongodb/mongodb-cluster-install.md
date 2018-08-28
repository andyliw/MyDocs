# MongoDB分片集群部署(整理)

URL:https://www.cnblogs.com/kaka171129/p/8024704.html

## 部署结构

### 1、集群结构

典型的三分片Mongo集群，包含三类组件：查询路由、配置存储、分片存储。

其中查询路由为mongos进程，配置存储和分片存储都是mongod进程。配置存储和分片存储都采取副本集（replicaset）来确保可用性和健壮性，每个副本集最少包含三个节点。查询路由都是单实例运行，没有副本集机制，可根据需要增加实例数量，也可以在外部添加负载均衡。

为了避免单点故障，同一个副本集中的成员，应该部署在不同主机上。

### 2、部署方案


### 3、部署环境

* 系统：CentOS7 64位
* 版本：mongodb-linux-x86_64-rhel70-3.6.6

## 部署步骤

## 安装MongoDB

### 1. 检查安装环境

* 检查 ulimit 配置

执行 `ulimit -a` ，查看当前设置，主要是确认最大文件数(-n)，最大进程数(-u)，官方推荐配置如下：

```
-f (file size): unlimited
-t (cpu time): unlimited
-v (virtual memory): unlimited
-n (open files): 64000
-m (memory size): unlimited
-u (processes/threads): 64000
```


执行 `vi /etc/security/limits.conf`，加入下面的配置：

```
# 最大文件数
* soft nofile 64000
* hard nofile 64000

# 最大进程数
* soft nproc 64000
* hard nproc 64000
```

### 2. 安装MongoDB

* 下载安装包

`http://downloads.mongodb.org/linux/mongodb-linux-x86_64-rhel70-3.6.6.tgz`

* 执行安装

```
tar -zxvf mongodb-linux-x86_64-rhel70-3.6.6.tgz
mv mongodb-linux-x86_64-rhel70-3.6.6.tgz /opt/mongodb
```

* 设置环境变更

```
vi ~/.bash.rc
export PATH=/opt/mongodb/bin:$PATH
```

* 验证程序

```shell
mongo --version
```

### 3. 安装numactl

只针对启用了NUMA的主机，如果无法通过BIOS关闭，可以安装numactl工具包。

* 下载安装包


* 执行安装指令

```
rpm -ivh numactl-xxxx.rpm
```

* 使用numactl

```
numactl --interleave=all mongod --config /var/mongodb/confsrv/mongod-cs.conf
```

### 4. 安装bwm-ng

* 下载安装包
* 执行安装指令

```
tar -zxvf bwm-ng.0.6.tar.gz
cd bwm-ng-0.6.1
./configure
make && make install
```

* 监控网络带宽

```
bwm-ng -u bits -d
bwm-ng -u bytes -d
```

* 监控磁盘IO

```
bwm-ng -u bytes -i disk
```

## 集群规划

### 1、节点分配

按三分片集群规划，它包含15个进程节点，3个路由（mongos）、1个配置存储副本集（3个mongod组成）、3个分片存储副本集（每个由3个mongod组成）。为了节省机器，按6台主机规划。

对资源消耗比较大的是分片存储节点（主节点或从节点），每台主机上最好只运行一个分片存储节点，采用一主一从一仲裁方式部署，6个分片存储节点平摊到每台主机上。

考虑到路由需要和配置服务器通信，所以路由和配置服务器两两部署在一起；另外路由与分片主节点通信也较多，所以就在每个分片主节点所在服务器上部署1个路由节点和1个配置服务器节点。

具体分配见下表：

|NO |主机|组件|
|:--|:---|:---|
|1  |192.168.10.101|shard1-primary, router, config-primary|
|2  |192.168.10.102|shard2-primary, router, config-secondary|
|3  |192.168.10.103|shard3-primary, router, config-secondary|
|4  |192.168.10.104|shard1-secondary, shard3-arbiter|
|5  |192.168.10.105|shard2-secondary, shard1-arbiter|
|6  |192.168.10.106|shard3-secondary, shard2-arbiter|

### 2、端口分配



### 3、路径和磁盘


### 4、集群规划清单

|组件   |角色     |主机          |端口 |目录                 |
|:------|:--------|:-------------|:----|:--------------------|
|confsrv|primary  |192.168.10.101|27019|/var/mongodb/confsrv/|
|confsrv|secondary|192.168.10.102|27019|/var/mongodb/confsrv/|
|confsrv|secondary|192.168.10.103|27019|/var/mongodb/confsrv/|
|shard1 |primary  |192.168.10.101|27018|/var/mongodb/shard1/ |
|shard1 |secondary|192.168.10.104|27018|/var/mongodb/shard1/ |
|shard1 |arbiter  |192.168.10.105|27019|/var/mongodb/shard1/ |
|shard2 |primary  |192.168.10.102|27018|/var/mongodb/shard2/ |
|shard2 |secondary|192.168.10.105|27018|/var/mongodb/shard2/ |
|shard2 |arbiter  |192.168.10.106|27019|/var/mongodb/shard2/ |
|shard3 |primary  |192.168.10.103|27018|/var/mongodb/shard3/ |
|shard3 |secondary|192.168.10.106|27018|/var/mongodb/shard3/ |
|shard3 |arbiter  |192.168.10.104|27019|/var/mongodb/shard3/ |
|router | --      |192.168.10.101|27017|/var/mongodb/router/ |
|router | --      |192.168.10.102|27017|/var/mongodb/router/ |
|router | --      |192.168.10.103|27017|/var/mongodb/router/ |

## 服务创建

### 1、创建配置存储服务

* 创建运行目录

```
mkdir -p /var/mongodb/confsrv/dbpath/
```

* 创建配置文件

```
# vi /var/mongodb/confsrv/mongod-cs.conf
sharding:
  clusterRole: configsvr
replication:
  replSetName: confsrv
  oplogSizeMB: 10240
systemLog:
  destination: file
  path: /var/mongodb/confsrv/mongod-cs.log
  logAppend: true
net:
  bindIp: 0.0.0.0
  port: 28019
storage:
  dbPath: /var/mongodb/confsrv/dbpath/
  directoryPerDB: true
processManagement:
  fork: true
  pidFilePath: /var/mongodb/confsrv/mongod-cs.pid
```

* 启动实例

```
mongod --config /var/mongodb/confsrv/mongod-cs.conf
```

* 初始化配置

确保三个配置存储实例都已正确启动，再通过 `mongo` 连接任一实例并执行以下指令:

```js
rs.initiate({
    _id: "confsrv",
    configsvr: true,
    members: [
        {_id: 0, host: "192.168.10.101:27019"},
        {_id: 1, host: "192.168.10.102:27019"},
        {_id: 2, host: "192.168.10.103:27019"}
    ]
});

// 管理指令
rs.conf();
rs.status();
```

### 2、创建分片存储服务

* 创建运行目录

```
mkdir -p /var/mongodb/shard1/dbpath/
```

* 创建配置文件

```
# vi /var/mongodb/shard1/mongod-sh.conf
sharding:
  clusterRole: shardsvr
replication:
  replSetName: shard1
  oplogSizeMB: 10240
systemLog:
  destination: file
  path: /var/mongodb/shard1/mongod-sh.log
  logAppend: true
net:
  bindIp: 0.0.0.0
  port: 28018
storage:
  dbPath: /var/mongodb/shard1/dbpath/
  directoryPerDB: true
processManagement:
  fork: true
  pidFilePath: /var/mongodb/shard1/mongod-sh.pid
```

* 启动实例

```
mongod --config /var/mongodb/shard1/mongod-sh.conf
```

* 初始化配置

确保三个配置存储实例都已正确启动，再通过 `mongo` 连接任一实例并执行以下指令:

```js
rs.initiate({
    _id: "shard1",
    members: [
        {_id: 0, host: "192.168.10.101:27018"},
        {_id: 1, host: "192.168.10.104:27018"}
    ]
});

// 增加仲裁节点
rs.addArb("192.168.10.105:27019");

// 管理指令
rs.conf();
rs.status();
```

### 3、创建查询路由服务

* 创建运行目录

```
mkdir -p /var/mongodb/router/
```

* 创建配置文件

```
# vi /var/mongodb/shard1/mongos.conf

sharding:
  configDB: confsrv/192.168.10.101:27019,192.168.10.102:27019,192.168.10.103:27019
systemLog:
  destination: file
  path: /var/mongodb/router/mongos.log
  logAppend: true
net:
  port: 27017
processManagement:
  fork: true
  pidFilePath: /var/mongodb/router/mongos.pid
```

* 启动实例

```
mongod --config /var/mongodb/router/mongos.conf
```

* 添加分片

通过 `mongo` 连接任一查询实例并执行以下指令:

```js
sh.addShard("shard1/192.168.10.101:27018");
sh.addShard("shard1/192.168.10.102:27018");
sh.addShard("shard1/192.168.10.103:27018");

// 管理指令
sh.status();
```

















