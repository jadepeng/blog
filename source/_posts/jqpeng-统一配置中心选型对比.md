---
title: 统一配置中心选型对比
tags: ["配置中心","jqpeng"]
categories: ["博客","jqpeng"]
date: 2018-08-24 11:03
---
文章作者:jqpeng
原文链接: [统一配置中心选型对比](https://www.cnblogs.com/xiaoqi/p/configserver-compair.html)

整理笔记时发现之前整理的一些东西，分享给大家。

## 为什么需要集中配置

**程序的发展，需要引入集中配置**：

- 随着程序功能的日益复杂，程序的配置日益增多：各种功能的开关、参数的配置、服务器的地址……
- 并且对配置的期望也越来越高，配置修改后实时生效，灰度发布，分环境、分集群管理配置，完善的权限、审核机制……
- 并且随着采用分布式的开发模式，项目之间的相互引用随着服务的不断增多，相互之间的调用复杂度成指数升高，每次投产或者上线新的项目时苦不堪言，因此需要引用配置中心治理。


**已有zookeeper、etcd还需要引入吗**？

- 之前的音乐服务项目，通过etcd实现了服务的注册与发现，且一些业务配置也存储到etcd中，通过实践我们收获了集中配置带来的优势
- 但是etcd并没有方便的UI管理工具，且缺乏权限、审核等机制
- 最重要的是，etcd和zookeeper通常定义为**服务注册中心**，统一配置中心的事情交给专业的工具去解决。


## 有哪些开源配置中心

1. spring-cloud/spring-cloud-config  
[https://github.com/spring-cloud/spring-cloud-config](https://github.com/spring-cloud/spring-cloud-config)  
 spring出品，可以和spring cloud无缝配合
2. 淘宝 diamond  
[https://github.com/takeseem/diamond](https://github.com/takeseem/diamond)  
 已经不维护
3. disconf  
[https://github.com/knightliao/disconf](https://github.com/knightliao/disconf)  
 java开发，蚂蚁金服技术专家发起，业界使用广泛
4. ctrip apollo  
[https://github.com/ctripcorp/apollo/](https://github.com/ctripcorp/apollo/)  
 Apollo（阿波罗）是携程框架部门研发的开源配置管理中心，具备规范的权限、流程治理等特性。


## 配置中心对别

### 功能特性

我们先从功能层面来对别


| **功能点** | **优先级** | **spring-cloud-config** | **ctrip apollo** | **disconf** | **备注** |
| --- | --- | --- | --- | --- | --- |
| 静态配置管理 | 高 | 基于file | 支持 | 支持 |  |
| 动态配置管理 | 高 | 支持 | 支持 | 支持 |  |
| 统一管理 | 高 | 无，需要github | 支持 | 支持 |  |
| 多环境 | 中 | 无，需要github | 支持 | 支持 |  |
| 本地配置缓存 | 高 | 无 | 支持 | 支持 |  |
| 配置锁 | 中 | 支持 | 不支持 | 不支持 | 不允许动态及远程更新 |
| 配置校验 | 中 | 无 | 无 | 无 | 如：ip地址校验，配置 |
| 配置生效时间 |  | 重启生效，或手动refresh生效 | 实时 | 实时 | 需要结合热加载管理， springcloudconfig需要 git webhook+rabbitmq 实时生效 |
| 配置更新推送 | 高 | 需要手工触发 | 支持 | 支持 |  |
| 配置定时拉取 | 高 | 无 | 支持 | 配置更新目前依赖事件驱动， client重启或者server端推送操作 |  |
| 用户权限管理 | 中 | 无，需要github | 支持 | 支持 | 现阶段可以人工处理 |
| 授权、审核、审计 | 中 | 无，需要github | 支持 | 无 | 现阶段可以人工处理 |
| 配置版本管理 | 高 | Git做版本管理 | 界面上直接提供发布历史和回滚按钮 | 操作记录有落数据库，但无查询接口 |  |
| 配置合规检测 | 高 | 不支持 | 支持（但还需完善） |  |  |
| 实例配置监控 | 高 | 需要结合springadmin | 支持 | 支持，可以查看每个配置在哪些机器上加载 |  |
| 灰度发布 | 中 | 不支持 | 支持 | 不支持部分更新 | 现阶段可以人工处理 |
| 告警通知 | 中 | 不支持 | 支持，邮件方式告警 | 支持，邮件方式告警 |  |
| 依赖关系 | 高 | 不支持 | 不支持 | 不支持 | 配置与系统版本的依赖系统运行时的依赖关系 |


### 技术路线兼容性

引入配置中心，需要考虑和现有项目的兼容性，以及是否引入额外的第三方组件。我们的java项目以SpringBoot为主，需要重点关注springboot支持性。


| **功能点** | **优先级** | **spring-cloud-config** | **ctrip apollo** | **disconf** | **备注** |
| --- | --- | --- | --- | --- | --- |
| SpringBoot支持 | 高 | 原生支持 | 支持 | 与spring boot无相关 |  |
| SpringCloud支持 | 高 | 原生支持 | 支持 | 与spring cloud无相关 |  |
| 客户端支持 | 低 | Java | Java、.Net | java |  |
| 业务系统侵入性 | 高 | 侵入性弱 | 侵入性弱 | 侵入性弱，支持注解及xml方式 |  |
| 依赖组件 | 高 | Eureka | Eureka | zookeeper |  |


### 可用性与易用性

引入配置中心后，所有的应用都需要依赖配置中心，因此可用性需要重点关注，另外管理的易用性也需要关注。


| **功能点** | **优先级** | **spring-cloud-config** | **ctrip apollo** | **disconf** | **备注** |
| --- | --- | --- | --- | --- | --- |
| 单点故障（SPOF） | 高 | 支持HA部署 | 支持HA部署 | 支持HA部署，高可用由zookeeper保证 |  |
| 多数据中心部署 | 高 | 支持 | 支持 | 支持 |  |
| 配置获取性能 | 高 | unkown | unkown（官方说比spring快） |  |  |
| 配置界面 | 中 | 无，需要通过git操作 | 统一界面（ng编写） | 统一界面 |  |


## 最终选择

综上，ctrip applo是较好的选择方案，最终选择applo。

- 支持不同环境（开发、测试、生产）、不同集群
- 完善的管理系统，权限管理、发布审核、操作审计
- SpringBoot集成友好 ，较小的迁移成本
- 配置修改实时生效（热发布）
- 版本发布管理


### 部署情况

- 管理Web：[http://config](http://config).\*\*\*.com/
- 三个环境MetaServer：
    - Dev： config.devmeta.\*\*\*.com
    - Test： config.testmeta.\*\*\*.com
    - PRO: config.prometa.\*\*\*.com


