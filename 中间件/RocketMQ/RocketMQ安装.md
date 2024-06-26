---
tags:
  - RocketMQ
  - 软件安装
---


RocketMQ是阿里巴巴2016年MQ中间件，使用Java语言开发，在阿里内部，RocketMQ承接了例如“双11”等高并发场景的消息流转，能够处理万亿级别的消息。

## 下载RocketMQ

官网：[https://rocketmq.apache.org/zh/](https://rocketmq.apache.org/zh/)

## 环境要求

* Linux64位系统
* JDK1.8(64位)
* 源码安装需要安装Maven

## 安装RocketMQ

以二进制包方式安装：

1. 解压安装包
2. 进入安装目录

### 目录介绍

* `bin`：启动脚本，包括shell脚本和CMD脚本
* `conf`：实例配置文件 ，包括broker配置文件、logback配置文件等
* `lib`：依赖jar包，包括Netty、commons-lang、FastJSON等

## 启动RocketMQ

> 如果启动失败：
> 1. 检查JDK的设置
> 2. RocketMQ 默认的虚拟机内存较大，启动Broker如果因为内存不足失败，需要编辑如下两个配置文件，修改JVM内存大小
>
> ```Bash
> # 编辑runbroker.sh和runserver.sh修改默认JVM大小
> vi runbroker.sh
> vi runserver.sh
> ```
> 参考设置：
> ```Bash
> JAVA_OPT="${JAVA_OPT} -server -Xms256m -Xmx256m -Xmn128m -XX:MetaspaceSize=128m  -XX:MaxMetaspaceSize=320m"
> JAVA_OPT="${JAVA_OPT} -XX:MaxDirectMemorySize=256m"
> ```

### 启动NameServer

```Bash
# 1.启动NameServer
nohup sh bin/mqnamesrv &
# 2.查看启动日志
tail -f ~/logs/rocketmqlogs/namesrv.log
```

### 启动Broker

```Bash
# 1.启动Broker
nohup sh bin/mqbroker -n localhost:9876 &
# 2.查看启动日志
tail -f ~/logs/rocketmqlogs/broker.log 
```

## 关闭RocketMQ

```Bash
# 1.关闭NameServer
sh bin/mqshutdown namesrv
# 2.关闭Broker
sh bin/mqshutdown broker
```

## 测试RocketMQ

### 发送消息

```Bash
# 1.设置环境变量
export NAMESRV_ADDR=localhost:9876
# 2.使用安装包的Demo发送消息
sh bin/tools.sh org.apache.rocketmq.example.quickstart.Producer
```

### 接收消息

```Bash
# 1.设置环境变量
export NAMESRV_ADDR=localhost:9876
# 2.接收消息
sh bin/tools.sh org.apache.rocketmq.example.quickstart.Consumer
```

