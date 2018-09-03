# 安装准备

## 一、集群规划

|IP             |System  |Hostname |User  |
|:--------------|:-------|:--------|:-----|
|192.168.10.157 |CentOS7 |hadoop1  |hadoop|
|192.168.10.158 |CentOS7 |hadoop2  |hadoop|
|192.168.10.160 |CentOS7 |hadoop3  |hadoop|

## 二、安装系统

### 1、安装CentOS7

安装CentOS7。

### 2、hosts设置

```
192.168.10.157  hadoop1
192.168.10.158  hadoop2
192.168.10.160  hadoop3
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

### 4、数据目录

```
mkdir -p /data
chown -R hadoop:hadoop /data
```

### 5、免密设置

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

## 三、安装依赖

### 1、ftp工具安装

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

### 2、Java安装

[详见](../centos7/java-install)

### 3、环境变量

```
export JAVA_HOME=/usr/java/jdk1.8.0_181-amd64
export CLASSPATH=.:$JAVA_HOME/lib:$JAVA_HOME/jre/lib
export PATH=JAVA_HOME/bin:$PATH
```


