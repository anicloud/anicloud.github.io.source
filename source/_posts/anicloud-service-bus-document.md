---
title: Ani.Cloud Service-Bus Document
date: 2016-10-18 21:23:21
categories: Ani.Cloud
tags: bus
top: true
---

## 概述
ServiceBus 为Octopus 核心提供对外的应用（服务）接入，以及基于OAuth2.0 的用户资源授信功能。是Anicloud 平台实现Device-to-Device，Service-to-Service 和Service-to-Device 的枢纽。 ServiceBus 对外提供平台用户资源、设备访问控制、对象状态管理、消息推送等核心功能。ServiceBus 的开放接口主要基于http 和WebSocket 协议。基于Http 协议的接口主要包括用户的资源、用户组资源、第三方服务注册、设备资源等。基于WebSocket 的开放接口主要实现Stub 的访问控制，对象状态维护等功能。
<!--more-->
## 模块列表
### 用户用户组模块
> 用户(Account)用户组(AccountGroup)模块提供用户的注册、基本信息修改以及用户组管理等功能。通过该模块可以实现基于用户组的授权功能，添加到目标用户所拥有的用户组的成员能够访问对应用户的资源（需要Octopus 提供支撑）。基于用户的多种信息进行查询。

### 第三方服务注册模块
> 第三方服务(AniService)模块提供第三方开发者在Anicloud 平台注册开发的应用的基本信息。经过平台验证后，第三方服务应用（或服务）便可以基于OAuth2.0 协议取得用户的授权(Token)，通过ServiceBus 提供的开放接口获取用户在平台上的资源。

> 第三方服务注册后，由平台办法唯一的ClientId 识别码。作为基于OAuth2.0 授信时的依据。


### 设备信息获取模块
> 设备信息获取模块提供对设备基本信息以及设备状态信息的获取功能。通过该模块第三方应用（或服务）可以获取对用户设备进行操作的Stub 列表，以及设备的状态信息。

### 基于OAuth2.0 的第三方应用授权
> ServiceBus 采用OAuth2.0 协议对用户的资源进行授权。基于Spring Security 和 Spring OAuth2.0 实现了对Authorization_Code，Implicit，Password 三种授权模式的支持。

## 设计实现
### 技术选择
* http，WebSocket，Jms
* Spring Framework，SpringMVC，Spring Security，Spring OAuth2.0
* Hibernate JPA，MySQL，Redis，ActiveMQ
* Maven，IDEA

### 系统设计
系统采用领域驱动（DDD）的设计方法。Spring 的注解@Configurable 提供了对领域层对象的注入需求，来实现对领域对象的装配。ServiceBus 分为Service-Bus模块和Service-Core模块。Service-Core 提供Bus内的核心业务，包括数据的持久化、与Octopus 系统的通信等。Service-Bus提供Bus对外的所有开放接口。从代码的组织结构上看分为：

* __interfaces__  接口层与外部系统通信
* __application__ 应用层，提供系统内的业务组装
* __domain__ 领域对方层，系统的核心功能
* __infrastructure__ 基础设施层，提供数据持久化，消息等功能

### 领域对象设计

  ![domain](/images/anicloud/service-bus-domain.png)

* __AniSerAccountObj__ 

  >记录用户、应用以及会话之间的状态。ServiceBus 的分布式部署特性，需要记录每一个第三方服务连接上Bus 的Session，以及通过该Session 与平台通信的Account 的状态。**objectId** 与 **token** 由Octopus 生成，是实现Bus-to-Octopus 安全访问的基础。

* __AniServiceDetails__

  >记录第三方应用在平台注册的基本信息。需要记录该应用的注册人信息(AccountId)。**aniServiceId** 与 **clientSecret** 字段由Bus 生成，作为第三方服务的识别码。同时该对象实现**org.springframework.security.oauth2.provider.ClientDetails** 接口。

* __AniService__

  >直接继承**AniServiceDetails** 类。作为Bus 上记录第三方服务的对象。

* __AniServiceInfo__

  >记录**AniService** 的非核信息，也为以后扩展AniService 信息的类。

* __AniServiceEntrance__

  >记录第三方服务的入口，即配置一个应用或服务的多个访问点。


### 核心对外接口 
* __基于http 协议的开放接口设计__  ServiceBus 采用SpringMVC 实现http 接口的发布，主要包括核心类：
  * AccountController，AccountGroupController

  ![Account-Controller](/images/anicloud/account-group-controller.png)

  * AniServiceController

  ![AniServiceController](/images/anicloud/aniservice-controller.png)

  * DeviceObjController
  
  ![DeviceObjController](/images/anicloud/deviceobj-controller.png)
  

* __基于WebSocket 设计的开放接口设计__
  * WebSocketServer 系统采用WebSocket 实现与第三方服务的全双工通信。WebSocket 通道主要实现对Object 的Stub 的同步/异步访问，Object的状态维护，消息的推送等功能。
  
  ![websocket](/images/anicloud/websocket.png)

### 核心业务类
* AniServiceManagerFacade 第三方服务核心业务类
  - getByAniServiceId(String aniServiceId)
  - saveOrUpdate(AniServiceDto aniServiceDto)
  - save(AniServiceDto aniServiceDto)
  - modify(AniServiceDto aniServiceDto)
  - removeByAniServiceId(String aniServiceId)
  - addAniServiceEntrance(String aniServiceId, AniServiceEntranceDto serviceEntranceDto)
  - addAniServiceEntrance(String aniServiceId, AniServiceEntranceDto serviceEntranceDto)

* AniSerAccountObjManager 应用、用户和会话状态维护核心类
  - registerAndLogin(AccountObject accountObject, String aniServiceId, String sessionId)
  - login(AccountObject accountObject, String aniServiceId, String sessionId)
  - remove(AccountObject accountObject, String aniServiceId, String sessionId)
  - updateStubList(AccountObject accountObject, String aniServiceId, String sessionId)
  - updateSessionRelatedAccountObjectState(String sessionId)
  - invokeObjectStub(AniStub aniStub, String aniServiceId, String sessionId) _Object的Stub调用_

* AniSerAccountObjServiceFacade 

 >AniSerAccountObj的增、删、改、查等基本操作类。
* CachedAniSerAccountObjServiceFacade 

 >提供AniSerAccountObj 的缓存操作的类，采用**装饰器模式** 包装了AniSerAccountObjServiceFacade 接口。

* DeviceObjInfoServiceFacade

 >提供对DeviceObj 的查询服务。

### 外部接口
ServiceBus 通过**antenna**模块封装的接口跟**Octopus** 核心通信。因此，需要实现antenna 的回调接口**ObjectInvokeListener**和**ObjectMessageListener**。

* ObjectInvokeHandler

 >实现ObjectInvokeListener 接口，提供对Object 的Stub 的调用。

* ObjectMessageHandler

 >实现ObjectMessageListener 的接口，提供消息的接收功能。

## 开发者文档
ServiceBus 提供了客户端**SDK** 开发者工具包，帮助第三方开发者快速地利用Anicloud 平台进行开发。详细参见 [Service-Agent](https://github.com/anicloud/octopus-object-client/wiki/Service-Bus-Agent-Docucment) 开发者文档。


## 部署要求
* 由于Spring 采用AOP 的方式实现DDD，为了让程序的war 包能够顺利发布到Tomcat 容器中，需要设置对应的环境变量。在tomcat 的bin 目录下的setenv.sh 文件中添加如下配置。如下图所示。

 ![tomcat](/images/anicloud/tomcat_config.png)

* tomcat 默认只支持50MB大小的文件上传，如果war 包过大需要修改tomcat 的配置文件。具体可网络搜索。

## 引用参考
* [Spring Document](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/)
* [Spring Security](http://projects.spring.io/spring-security/)
* [OAuth2.0 规范](https://github.com/jeansfish/RFC6749.zh-cn/blob/master/TableofContents.md)

