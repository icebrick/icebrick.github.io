---
title: apache storm相关概念备忘
key: 20190512
tags: storm。
---

使用apache storm框架过程中了解到的storm集群相关概念记录下来备忘

<!--more-->

#### daemon：

- 命令：`daemon -r -c /path_to_storm/bin/storm supervisor`
- 当storm进程挂了，daemon会自动把它拉起来
- 当daemon进程挂了，storm进程还会继续，但是如果之后storm进程挂了，就不会再被拉起

#### storm集群需要：

- zookeeper集群
- nimbus
- supervisor

#### nimbus：

- 负责协调整个集群
- 整个集群一般只需要一个

#### supervisor：

- 在每台工作机上需要一个supervisor进程
- supvisor进程负责管理该机器上的所有worker
- supervisor主进程会为每一个worker(used slot) fork一个子进程来管理该worker，如果worker进程挂了，负责将其拉起来
- 当supervosr和daemon都挂掉之后，worker进程不受影响，继续服务，只是如果这时worker挂了之后就不会被拉起了，新的worker也不会被分配到这台机器

#### zookeeper：

- /storm/supervisor下保存目前工作中的supervisor节点。有新的supervisor节点加入集群，会将自己的id和相关信息写入到这个路径（作为临时节点）；有节点下线，对应的信息会自动被删除（zookeeper检测到对应的客户端失联）

#### 增加集群节点流程
- 使用和集群中其他节点一样的storm配置
- 使用守护程序启动supervisor
- 有必要的话可以对已有的topology rebalance

#### 下线集群中的节点
- kill掉守护supervisor的守护进程
- kill掉supervisor进程
- kill所有的worker
- 或者直接shut down这台机器