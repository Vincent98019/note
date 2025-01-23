
## 什么是Redis

Redis是用C语言开发的一个开源的高性能键值对（key-value）数据库，Redis通过提供多种键值数据类型来适应不同场景下的存储需求。

目前为止Redis支持的键值数据类型如下：

- 字符串类型 string
- 哈希类型 hash
- 列表类型 list
- 集合类型 set
- 有序集合类型 sortedset

这些数据类型都支持push/pop、add/remove及取交集并集和差集及更丰富的操作，而且这些操作都是**原子性**的，并支持各种不同方式的**排序**。

Redis数据是缓存在内存中，会周期性的把更新的数据写入磁盘或者把修改操作写入追加的记录文件。并且在此基础上实现了master-slave(主从)同步。

Redis是单线程+多路IO复用技术

> 多路复用是指使用一个线程来检查多个文件描述符（Socket）的就绪状态，比如调用select和poll函数，传入多个文件描述符，如果有一个文件描述符就绪，则返回，否则阻塞直到超时。得到就绪状态后进行真正的操作可以在同一个线程里执行，也可以启动线程执行（比如使用线程池）。
> 

## Redis的应用场景

- 配合关系型数据库做高速缓存（数据查询、短连接、新闻内容、商品内容等等）
- 聊天室的在线好友列表
- 任务队列（秒杀、抢购、12306等等）
- 应用排行榜
- 网站访问统计
- 数据过期处理（可以精确到毫秒）
- 分布式集群架构中的session分离

## Redis安装

- 官网：[https://redis.io](https://redis.io/)
- 中文网：[http://www.redis.net.cn](http://www.redis.net.cn/)

这里安装 `6.2.1 for Linux` 版本

**安装GCC**

```bash
yum install centos-release-scl scl-utils-build
yum install -y devtoolset-8-toolchain
scl enable devtoolset-8 bash
```

**查看GCC版本**

```bash
gcc --version
```

下载 `redis-6.2.1.tar.gz` 放 `/opt` 目录

解压，并进入文件夹

```bash
tar -zxvf redis-6.2.1.tar.gz
cd redis-6.2.1
```

在 redis-6.2.1 目录下执行命令：

```bash
make
make install
```

如果没有准备好C语言编译环境，`make`会报错：`Jemalloc/jemalloc.h：没有那个文件` ，这时执行`make distclean` 命令后再执行`make` 命令。

进入安装目录：

```bash
cd /usr/local/bin
```

- `redis-benchmark` ：性能测试工具
- `redis-check-aof` ：修复有问题的AOF文件
- `redis-check-dump` ：修复有问题的dump.rdb文件
- `redis-sentinel` ：Redis集群使用
- `redis-server` ：Redis服务器启动命令
- `redis-cli` ：客户端，操作入口

## Redis常用命令

### 前台启动（不推荐）

```bash
redis-server
```

### 后台启动（推荐）

备份 `redis.conf` 文件，拷贝一份到其他目录

```bash
mkdir /home/data
mkdir /home/data/redis
cp /opt/redis-6.2.1/redis.conf /home/data/redis
```

修改 `redis.conf` (128行)文件将里面的 `daemonize no` 改成 `yes`，让服务在后台启动。

![](assets/Redis概述&安装/9e0b0d70bcd83ca75579b7af2340c6ce_MD5.png)


启动Redis：

```
redis-server /home/data/redis/redis.conf
```

### 客户端

```bash
redis-cli
```

指定端口：

```bash
redis-cli -p 6379
```

### 关闭Redis

单实例关闭：

```bash
redis-cli shutdown
```

指定端口关闭：

```bash
redis-cli -p 6379 shutdown
```

进入Redis后关闭：

```bash
shutdown
```

![](assets/Redis概述&安装/2a2ad68b6e46df52da6851f367f100b1_MD5.png)
