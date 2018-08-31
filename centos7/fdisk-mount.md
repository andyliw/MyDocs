# 华为云磁盘挂载

## 查看

```
fdisk -l
```

## 分区

```
fdisk /dev/vdb

# 指令列表：n | p | 1 | 1 |

# m: 显示所有命令列表
# p: 显示硬盘分割信息，打印分区表
# a: 设定硬盘启动区
# n: 设定新的硬盘分割区
# n > e: 硬盘为延伸分割区
# n > p: 硬盘为主要分割区
# t: 改变硬盘分割区属性
# d: 删除硬盘分割信息
# q: 结束不写入硬盘分割信息
# w: 结束并写入硬盘分割信息
```

## 格式化

```
mkfs.[fmt] /dev/vdb1

# fmt: 文件系统格式(ext3|ext4|xfs)
```

## 挂载

```
mount /dev/vdb1 /data
```

## 

```
# blkid
# vi /etc/fstab

-----------------

UUID=xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx  /data  xfs  defaults  2 1

```
