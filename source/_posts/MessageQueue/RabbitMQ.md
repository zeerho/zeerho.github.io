---
title: RabbitMQ
date: 2018-04-11 12:00:00
tags: [MQ]
---

# vhost

进入 RabbitMQ 安装路径。

- 创建: `rabbitmqctl add_vhost {vhost_name}`
- 删除: `rabbitmqctl delete_vhost {vhost_name}`
- 列举: `rabbitmqctl list_vhosts` 
- 管理远程节点: `rabbitmqctl -n {erl_app}@{server_name}`
    - {erl_app} 是 erlang 应用程序名字，这里固定为 rabbit。
    - {server_name} 是服务器主机名或 IP 地址。
    - 需要确保运行 Rabbit 节点的服务器和运行 rabbitmqctl 的工作站安装了相同的 Erlang Cookie。
