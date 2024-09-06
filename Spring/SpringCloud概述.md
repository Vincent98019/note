
## 微服务

随着互联网行业的发展，对服务的要求也越来越高，服务架构也从单体架构逐渐演变为现在流行的微服务架构。

![](assets/SpringCloud概述/9X4fyl3vh2L-ioqzg0RZjc3TzfyHWoNvGhtZ32k13xk.png)


微服务这种方案需要技术框架来落地，全球的互联网公司都在积极尝试自己的微服务落地技术。在国内最知名的就是SpringCloud和阿里巴巴的Dubbo。

**单体架构**：将业务的所有功能集中在一个项目中开发，打成一个包部署。

**优点：**

* 架构简单
* 部署成本低

**缺点：**

* 耦合度高（维护困难、升级困难）


![](assets/SpringCloud概述/lEHyU5kOVB5CWILVBFBBkWi9383kpaDvTLu5W_yuufw.png)


**分布式架构**：根据业务功能对系统做拆分，每个业务功能模块作为独立项目开发，称为一个服务。

**优点：**

* 降低服务耦合
* 有利于服务升级和拓展

**缺点：**

* 服务调用关系错综复杂


![](assets/SpringCloud概述/-dqX7GtNHMacrVC0xXRFMo0QTH9x96Nvyj1_LXZ3XpY.png)



微服务的架构特征：

* 单一职责：微服务拆分粒度更小，每一个服务都对应唯一的业务能力，做到单一职责
* 自治：团队独立、技术独立、数据独立，独立部署和交付
* 面向服务：服务提供统一标准的接口，与语言和技术无关
* 隔离性强：服务调用做好隔离、容错、降级，避免出现级联问题

![](assets/SpringCloud概述/t_Qj72zLJhjrs5IsxdEY8RopaXhelNH48NERuUmWiyI.png)

微服务的上述特性其实是在给分布式架构制定一个标准，进一步降低服务之间的耦合度，提供服务的独立性和灵活性。做到高内聚，低耦合。

因此，可以认为**微服务**是一种经过良好架构设计的**分布式架构方案** 。


## 微服务技术对比

![](assets/SpringCloud概述/pJOviPCVtELO_gLIpwpwDPg7H10RecBuJXTjNaIVxnc.png)

## SpringCloud

SpringCloud是目前国内使用最广泛的微服务框架。官网地址：[https://spring.io/projects/spring-cloud](https://spring.io/projects/spring-cloud)。

SpringCloud集成了各种微服务功能组件，并基于SpringBoot实现了这些组件的自动装配，从而提供了良好的开箱即用体验。

其中常见的组件包括：

![](assets/SpringCloud概述/7ieP3CsRnSJ_IsGRHwJE1cbIRQe4KfzOi3L62FdWfdY.png)

SpringCloud底层是依赖于SpringBoot的，并且有版本的兼容关系，如下：

![](assets/SpringCloud概述/Io7CQfRRESkhcOrOwPByS_QfKEMaa6L2tpIJ6k1HA0Y.png)

* 单体架构：简单方便，高度耦合，扩展性差，适合小型项目。例如：学生管理系统
* 分布式架构：松耦合，扩展性好，但架构复杂，难度大。适合大型互联网项目，例如：京东、淘宝
* 微服务：一种良好的分布式架构方案
   * 优点：拆分粒度更小、服务更独立、耦合度更低
   * 缺点：架构非常复杂，运维、监控、部署难度提高
* SpringCloud是微服务架构的一站式解决方案，集成了各种优秀微服务功能组件













