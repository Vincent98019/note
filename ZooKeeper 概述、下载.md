---
tags:
- ZooKeeper
---


Zookeeper 是一个开源的分布式的，为分布式框架提供协调服务的 Apache 项目。
![](assets/ZooKeeper%20概述、下载/image-20240429154034960.png)

## Zookeeper工作机制

Zookeeper从设计模式角度来理解：是一个基于观察者模式设计的分布式服务管理框架，它负责存储和管理大家都关心的数据，然后接受观察者的注册，一旦这些数据的状态发生变化，Zookeeper就将负责通知已经在Zookeeper上注册的那些观察者做出相应的反应。

![](assets/ZooKeeper%20概述、下载/image-20240429154100509.png)

## Zookeeper特点

![](assets/ZooKeeper%20概述、下载/image-20240429154122891.png)

- Zookeeper：一个领导者（Leader），多个跟随者（Follower）组成的集群。
- 集群中只要有半数以上节点存活，Zookeeper集群就能正常服务。所以Zookeeper适合安装奇数台服务器。
- 全局数据一致：每个Server保存一份相同的数据副本，Client无论连接到哪个Server，数据都是一致的。
- 更新请求顺序执行，来自同一个Client的更新请求按其发送顺序依次执行。
- 数据更新原子性，一次数据更新要么成功，要么失败。
- 实时性，在一定时间范围内，Client能读到最新数据。


## 数据结构

ZooKeeper 数据模型的结构与 Unix 文件系统很类似，整体上可以看作是一棵树，每个节点称做一个 ZNode。每一个 ZNode 默认能够存储 1MB 的数据，每个 ZNode 都可以通过其路径唯一标识。

![](assets/ZooKeeper%20概述、下载/image-20240429154141569.png)


## 应用场景

提供的服务包括：统一命名服务、统一配置管理、统一集群管理、服务器节点动态上下线、软负载均衡等。

### 统一命名服务

在分布式环境下，经常需要对应用/服务进行统一命名，便于识别。

例如：IP不容易记住，而域名容易记住。

![](assets/ZooKeeper%20概述、下载/image-20240429154157556.png)


### 统一配置管理

**分布式环境下，配置文件同步非常常见。**

- 一般要求一个集群中，所有节点的配置信息是一致的，比如 Kafka 集群。
- 对配置文件修改后，希望能够快速同步到各个节点上。

**配置管理可交由ZooKeeper实现。**

- 可将配置信息写入ZooKeeper上的一个Znode。
- 各个客户端服务器监听这个Znode。
- 一旦Znode中的数据被修改，ZooKeeper将通知各个客户端服务器。

![](assets/ZooKeeper%20概述、下载/image-20240429154213908.png)


### 统一集群管理

**分布式环境中，实时掌握每个节点的状态是必要的。**

- 可根据节点实时状态做出一些调整。

**ZooKeeper可以实现实时监控节点状态变化。**

- 可将节点信息写入ZooKeeper上的一个ZNode。
- 监听这个ZNode可获取它的实时状态变化。

![](assets/ZooKeeper%20概述、下载/image-20240429154233085.png)


### 服务器动态上下线

客户端能实时洞察到服务器上下线的变化。

![](assets/ZooKeeper%20概述、下载/image-20240429154248910.png)


### 软负载均衡

在Zookeeper中记录每台服务器的访问数，让访问数最少的服务器去处理最新的客户端请求。

![](assets/ZooKeeper%20概述、下载/image-20240429154306828.png)


## 五、下载
官网： [http://zookeeper.apache.org/](http://zookeeper.apache.org/)

![](assets/ZooKeeper%20概述、下载/image-20240429154331861.png)

