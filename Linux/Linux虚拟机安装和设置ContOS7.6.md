


## 安装系统

> 选择**中文-简体中文 (中国)**，继续

![image-20221021174340504](https://img-blog.csdnimg.cn/img_convert/71300293ff385ccd5ef380def7a99f36.png)



- **软件选择**

  > 点击**软件选择**
  >
  > 一般生产环境选择 **最小安装**，里面什么都没有，而学习阶段选择 **GNOME 桌面版**，选中：
  >
  >  - [x] **传统 X Windows 系统的兼容性**
  >  - [x] **兼容性程序库**
  >  - [x] **开发工具**
  >
  > 开发工具中包含了JDK，GCC，MySQL等等，不用自己再安装了，当然后续可以卸载或者更新
  >
  > 然后点击**完成**

![image-20221021174513935](https://img-blog.csdnimg.cn/img_convert/02b1827245245f28a882c467452886a4.png)

![image-20221021174555934](https://img-blog.csdnimg.cn/img_convert/d795563f38d2b3356863659184a509cf.png)



- **安装位置**

  > 点击完成后，会出现**安装源**和**软件选择**检查软件依赖关系的等待时间，等待期间不要乱动，等待结束后，点击下面的**安装位置**。
  >
  > 一般选择**自动配置分区**，为了更能熟悉Linux系统，这里选择**我要配置分区**，点击**完成**。

![image-20221021175051961](https://img-blog.csdnimg.cn/img_convert/ea9be98d85a7b22c38ae6f76800d38be.png)



> 上面配置内存的时候，给了虚拟机20G的内存，而自定义分区的话，一般分成三个区
>
> 1. **boot分区**：引导分区，操作系统的内核及在引导过程中使用的文件，**建议给1G**
> 2. **swap分区**：交换分区，当实际内存不够用的时候，操作系统会从内存中取出一部分暂时不用的数据，放在交换分区中，从而为当前运行的程序腾出足够的内存空间，**建议跟内存大小一样2G**
> 3. **根分区**：就是root分区，**剩下17G都给根分区**
>
> 点击 **+** 号，添加分区

![image-20221021175222432](https://img-blog.csdnimg.cn/img_convert/701853c6d3227dbd6655124be14e2643.png)



> **挂载点**选择 **/boot**，**期望容量**输入**1G**，点击**添加挂载点**后，将**设备类型**改成**标准分区**，**文件系统**改成**ext4**

![image-20221021175301727](https://img-blog.csdnimg.cn/img_convert/92673594cf5bce9b41b0dbee88fc1244.png)

![image-20221021175352912](https://img-blog.csdnimg.cn/img_convert/ad6f340e7e1a4070e9f847fe8871ca4c.png)



> 同样方式创建**swap分区(挂载点选择swap，期望容量输入2G)**，将**设备类型**改成**标准分区**，**文件系统**改成**swap**
>
> 创建**根分区(挂载点选择/，期望容量输入17G)**，将**设备类型**改成**标准分区**，**文件系统**改成**ext4**
>
> 点击左上角**完成**后，点击**接受更改**

![image-20221021175527450](https://img-blog.csdnimg.cn/img_convert/cc6ba02c7a265211615a31a34d6cf57d.png)



- **KDUMP**

  > 用于系统崩溃时捕获信息，如果是生产环境建议**勾选**，如果是学习的话可以**关掉**，节省内存。

![image-20221021175659270](https://img-blog.csdnimg.cn/img_convert/06167cff7399c8a21c5f8badd722168d.png)



- **网络和主机名**

  > 选择**网络和主机名**
  >
  > **更改主机名**后点**应用**，将右上角的**以太网**选项打开，表示联网，最后点击**完成**

![image-20221021175846264](https://img-blog.csdnimg.cn/img_convert/329ee79fe5e59f1111979c309a861f23.png)



- **SECURITY POLICY**

  > 安全策略，用于密码的校验等等，可以根据需求关闭，这里我选择关闭

![image-20221021175929278](https://img-blog.csdnimg.cn/img_convert/1222276b583268a97bbcfb4119a2ae2e.png)





> 点击**开始安装**

![image-20221021180006656](https://img-blog.csdnimg.cn/img_convert/e65494fef3f33e9ba78cdc51c4eafa55.png)



> 设置**root密码**

![image-20221021180028723](https://img-blog.csdnimg.cn/img_convert/86d6d94dce279e8a4cb5b2a648576776.png)

![image-20221021180225303](https://img-blog.csdnimg.cn/img_convert/6e3ae6017aa0ff51049c12beb183765f.png)



> 一般不用root登录，因为权限太大了，所以点击**创建用户**，然后点击**完成**

![image-20221021180249105](https://img-blog.csdnimg.cn/img_convert/41652d4e80d4e0424200e4f3b3304264.png)

![image-20221021180335111](https://img-blog.csdnimg.cn/img_convert/6187cc89e2b86aef281372b63d5c46e7.png)



> 配置完成后，等待一段时间安装，安装成功后，点击**重启**

![image-20221021181049989](https://img-blog.csdnimg.cn/img_convert/54c35ea4d427c20c07b1db56aaa284ae.png)



> 重启后，点击**LICENSE INFORMATION**，然后选择**我同意许可协议**，再点击**完成**，出来后点击**完成配置**

![image-20221021181141890](https://img-blog.csdnimg.cn/img_convert/81dddecf4c6a010794889b98520764a4.png)



> Linux默认情况下希望使用普通用户登录，如果需要使用root用户登录，可以点击**未列出**，输入**用户名**和**密码**然后点击**登录**即可，第一次可能需要等待时间较长

![image-20221021181252057](https://img-blog.csdnimg.cn/img_convert/15d9538231e768d414261b05cd970c94.png)

![image-20221021181306391](https://img-blog.csdnimg.cn/img_convert/9278ee96533b824ac36d674ba6e62a61.png)

![image-20221021181325017](https://img-blog.csdnimg.cn/img_convert/ce2cea3c93bbf13308001517605ccebd.png)



> 1. 选择**汉语**，点击**前进**
>
> 2. 选择**汉语拼音**，点击**前进**
>
> 3. 位置服务可以选择**关闭**，点击**前进**
> 4. 登录账号，这里可以点击**跳过**
> 5. 配置和安装完成

![image-20221021181404087](https://img-blog.csdnimg.cn/img_convert/1d235e3f4abd919148c8bafbc52e3da1.png)

![image-20221021181434824](https://img-blog.csdnimg.cn/img_convert/df56ca6afef087bd74e4f14d7b04d5c3.png)

![image-20221021181508767](https://img-blog.csdnimg.cn/img_convert/079e00b40393fb5a1d3c8b1d9e608851.png)

![image-20221021181547799](https://img-blog.csdnimg.cn/img_convert/df2476cbebd6177d93f3547b1be0a79a.png)