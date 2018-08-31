# RabbitMQ安装

## 系统环境

CentOS 7

## Erlang 安装

* 下载

访问`https://github.com/rabbitmq/erlang-rpm/releases`，下载最新的安装包。
例如：`erlang-21.0.6-1.el7.centops.x86_64.rpm`

* 安装

```
rpm -ivh erlang-21.0.6-1.el7.centops.x86_64.rpm
```

## RabbitMQ 安装

* 下载

访问`https://github.com/rabbitmq/rabbitmq-server/releases`，下载最新的安装包。
例如：`rabbitmq-server-3.7.8.rc.3-1.el7.noarch.rpm`

* 安装

```
rpm -ivh rabbitmq-server-3.7.8.rc.3-1.el7.noarch.rpm
```

## 延时队列

* 下载

访问`http://www.rabbitmq.com/community-plugins.html`，下载最新的安装包。
例如：`rabbitmq_delayed_message_exchange-20171201-3.7.x.ez`

* 安装

```shell
cp rabbitmq_delayed_message_exchange-20171201-3.7.x.ez /usr/lib/rabbitmq/lib/rabbitmq_server-3.7.5/plugins

rabbitmq-plugins enable rabbitmq_delayed_message_exchange
```

## RabbitMQ 配置

```
rabbitmqctl list_users

rabbitmqctl add_user ygxt ygyg2017
rabbitmqctl set_user_tags ygxt administrator
rabbitmqctl set_permissions -p /vhosts1 ygxt '.*' '.*' '.*'
```

