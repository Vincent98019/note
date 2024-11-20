
## 下载&安装

官方下载地址：[https://www.rabbitmq.com/download.html](https://www.rabbitmq.com/download.html)

在安装之前需要先安装Erlang环境

![](assets/RabbitMQ安装/27f8b9dadd3c46d9724a45971480fc84_MD5.png)


上传到`/usr/local/software`目录下(如果没有 software 需要自己创建)，执行`uname -a`可以查看当前系统版本是el7，和下载的安装包是符合。
![](assets/RabbitMQ安装/a9a9101a6349e1254e2bb5d76da098be_MD5.png)




安装命令，按顺序安装：

```bash
rpm -ivh erlang-21.3-1.el7.x86_64.rpm
yum install socat -y
rpm -ivh rabbitmq-server-3.8.8-1.el7.noarch.rpm
```


## 常用命令

### 添加开机启动 RabbitMQ 服务

```bash
chkconfig rabbitmq-server on
```


### 启动服务

```bash
/sbin/service rabbitmq-server start
```


### 查看服务状态

```bash
/sbin/service rabbitmq-server status
```
![](assets/RabbitMQ安装/2a1fc34ef84c21eea3e3eb09ff67dd51_MD5.png)




### 停止服务

```bash
/sbin/service rabbitmq-server stop
```


### 开启 web 管理插件

需要先停止rabbitmq服务，进入rabbitmq的安装目录，然后执行下面命令，再重新运行rabbitmq服务。

```bash
rabbitmq-plugins enable rabbitmq_management
```

开启服务后，进入web界面，地址是http://ip:15672



#### 问题解决

**如果进不去的话，可以看下防火墙有没有关掉。**

* 关闭防火墙：

```bash
systemctl stop firewalld.service
```

* 查看防火墙状态：

```bash
systemctl status firewalld.service
```

* 下次开机也关闭防火墙：

```bash
systemctl disable firewalld.service
```

**如果还是进不去，进入rabbitmq的安装目录的bin目录中执行命令：**

1. 先查看rabbitmq的安装目录，可以先进入第一个文件夹中看看是否安装到该目录(看下该目录中是否有bin目录)

```bash
whereis rabbitmq
```

![](assets/RabbitMQ安装/28bcf4e020c918a9a7564286d9222e56_MD5.png)


2. 进入安装目录后执行下面命令：

```bash
rabbitmq-plugins enable rabbitmq_management
```

3. 执行后，重启rabbitmq服务（先停止，再启动，然后查看状态是否运行）：

```bash
/sbin/service rabbitmq-server stop
/sbin/service rabbitmq-server start
```

4. 此时进入web界面即可看到内容

![](assets/RabbitMQ安装/bcddfd12aa61bc7e434888d269f29735_MD5.png)




## 设置权限

使用默认账号密码(guest)登录，会发现登录不上，该账号只支持本地登录，所以需要设置新的账号和设置权限。

![](assets/RabbitMQ安装/5cbb8877d2ba27ad25ac4f39ae4b9216_MD5.png)




### 创建账号

```bash
rabbitmqctl add_user 用户名 密码
```

![](assets/RabbitMQ安装/e9719aabb74bce6a6999b424ad8d939e_MD5.png)




### 设置用户角色

```bash
rabbitmqctl set_user_tags vh 用户名 权限(administrator)
```

![](assets/RabbitMQ安装/5cbfbb0d8c074453ef337ab7c52ec62e_MD5.png)




### 设置用户权限

```bash
rabbitmqctl set_permissions [-p <vhostpath>] <user> <conf> <write> <read>

# 案例：用户 admin 具有/vhost1 这个 virtual host 中所有资源的配置、写、读权限
rabbitmqctl set_permissions -p "/" admin ".*" ".*" ".*"
```

![](assets/RabbitMQ安装/45298b0769ecfcb18a98a15374a595f6_MD5.png)




### 查询用户和角色

```bash
 rabbitmqctl list_users
```

![](assets/RabbitMQ安装/7d2882de24e3926d467fa24bc338edd0_MD5.png)




### 再次登录

后续可以在web界面添加账户

![](assets/RabbitMQ安装/dc369814610bac7139659485f80aae38_MD5.png)




### 重置命令

#### 关闭应用

```bash
rabbitmqctl stop_app
```

#### 清除

```bash
rabbitmqctl reset
```

#### 重新启动

```bash
rabbitmqctl start_app
```




## Docker安装

拉取镜像

```bash
docker pull rabbitmq:3-management
```


创建并运行容器：

```bash
docker run \
 -e RABBITMQ_DEFAULT_USER=admin \
 -e RABBITMQ_DEFAULT_PASS=123123 \
 --name mq \
 --hostname mq1 \
 -p 15672:15672 \
 -p 5672:5672 \
 -d \
 rabbitmq:3-management
```
