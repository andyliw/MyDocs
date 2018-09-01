# Java SDK安装

## 卸载OpenJDK

指令：

```
rpm -qa | grep java
rpm -e --nodes java-1.8.0-openjdk-1.8.0.161-2.b14.el7.x86_64
rpm -e --nodes java-1.8.0-openjdk-headless-1.8.0.161-2.b14.el7.x86_64
```

日志：

```
[root]# rpm -qa | grep java
java-1.8.0-openjdk-1.8.0.161-2.b14.el7.x86_64
tzdata-java-2018c-1.el7.noarch
python-javapackages-3.4.1-11.el7.noarch
javapackages-tools-3.4.1-11.el7.noarch
java-1.8.0-openjdk-headless-1.8.0.161-2.b14.el7.x86_64
[root]# rpm -e --nodes java-1.8.0-openjdk-1.8.0.161-2.b14.el7.x86_64
[root]# rpm -e --nodes java-1.8.0-openjdk-headless-1.8.0.161-2.b14.el7.x86_64
[root]# rpm -qa | grep java
```

## 安装Java

下载JDK的rpm安装包。

```
rpm -ivh jdk-8u181-linux-x64.rpm
```
