---
title: Ani.Cloud Ani-Agent Document
date: 2016-10-18 21:33:42
categories: Ani.Cloud
tags: agent
top: true
---


## 概述
**Ani-Agent**模块是为第三方开发者提供的SDK。该模块封装了**Service-Bus**对外提供的开放接口的常用服务。利用该模块，开发者可以方便的进行第三方服务的注册、用户用户组管理、基于OAuth2.0授信、设备对象信息获取以及对象Stub调用等功能。
<!--more-->
## 模块列表
### 用户用户组
> 该模块提供对用户用户组的基本操作。包括用户的注册、修改；以及用户组的添加、删除、关联等操作。

### 第三方服务注册
> 该模块提供第三方服务的注册功能。开发者能够提供该模块在Anicloud 平台注册自己的应用（服务）。平台会为该应用颁发唯一的ID以及client_secret。

### 设备对象（DeviceObj）信息获取
> 该模块提供对DeviceObj的信息查询功能。通过用户授信的第三方应用可以利用该接口获取用户的设备状态信息、设备Stub 列表信息等。

### 基于OAuth2.0 的授信
> 该模块提供对OAuth2.0 协议的流程封装，提供该模块可以方便地进行access_token的获取以及token的刷新操作。该模块提供了对**Authorization Code**、**Password**、**Implicit**三种授信模式的支持。

### 对象Stub调用
> 该模块提供了对象Stub调用的接口**AniInvokable**。对Stub的调用提供了同步和异步两种方式。异步的调用需要开发者实现消息结果解析的观察者接口**MessageObserver**，利用该接口来处理异步调用的结果。同时也提供了平台对应用的反向调用接口**ClientInvokable**，该接口需要由应用的开发者实现。

## 设计实现

### 技术选择
* 协议WebSocket、Http、OAuth2.0
* 设计模式：观察者、工厂、单例
* Spring RestTemplate 提供对Http 接口的访问

### 系统配置
anicel_meta.properties 文件是**Servuce-Agent**的核心配置文件。主要包括**ServiceBus**的各类资源地址。当**ServiceBus**的地址发生改变时，可以对该文件进行编辑。

### 核心类
* __AnicelMeta__ 提供对anicel_meta.properties 文件的读取操作，并作为**Service-Agent**的核心配置类存在。Http和Websocket的实现都依赖于该类。
* __DomainObjectValidator__ 提供对Bean Valication的实现。采用的是Hibernate Validation 实现。
* __AccountObject__ 该类提供对用户状态的维护功能，同时第三方应用通过该类，为用户对象绑定Stub列表。以便实现对用户Stub的调用。

### 核心业务类
* __AccountService__ 该接口提供对用户的基本操作。需要注意的是注册用户时，不需要提供token。
  * register(AccountRegisterDto account) 
  * modify(AccountModifyDto account)
  * getByAccountId(Long accountId)
  * getByEmail(String email)
  * getByPhoneNumber(String phoneNumber)
  * getByAccessToken()
  * addAccountInGroup(Long accountId, Long groupId) 添加用户到用户组
  
* __AccountGroupService__ 该接口提供对用组的基本操作。
  * save(GroupFormDto accountGroup)
  * modify(GroupFormDto accountGroup)
  * remove(Long accountId, Long groupId)
  * getByAccountAndGroupType(Long accountId, GroupType groupType)
  * getAccountsInGroup(Long groupId)

* __AniServiceManager__ 该接口提供第三方开发者注册应用的操作。注册应用时不需要提供token。
  * register(AniServiceRegisterDto registerDto)
  * getByAniService(String aniServiceId, String clientSecret)

* __DeviceObjService__ 该接口提供对设备对象的获取操作。
  * getDeviceObjInfo(Long accountId, boolean withSlave)
  * getDeviceObjInfo(Long accountId, Long mainObjId, boolean withSlave)
  
* __AniOAuthService__ 该接口实现了OAuth2.0 的相关流程。
  * getOAuth2AccessToken(String code, AuthorizationCodeParameter authorizationCodeParameter)
  * getOAuth2AccessToken(PasswordParameter passwordParameter)
  * getOAuth2AccessToken(ImplicitParameter implicitParameter) 暂未实现
  * refreshAccessToken(String refreshToken, AuthorizationCodeParameter authorizationCodeParameter)

* __AccountInvoker__ 该接口是第三方应用维护用户对象状态（AccoutObject）的接口。Anicloud 平台需要维护Account-App-ServiceBus的状态信息。需要第三方应用在适当的地方调用该接口的实现类，向平台汇报用户状态。
  * registerAndLogin(AccountObject accountObject)
  * login(AccountObject accountObject)
  * logout(AccountObject accountObject) 
  * remove(AccountObject accountObject)
  * updateAccountObjectStubList(AccountObject accountObject)
  
* __AniInvokable__ 该接口提供对对象Stub的同步和异步调用。当调用为异步调用时，需要注册实现**MessageObserver**接口的消息处理观察者到WebsocketClient 对象中。
  * invokeAniObjectAsyn(AniStub stub) 
  * invokeAniObjectSync(AniStub stub) 
  
* __ClientInvokable__ 该接口是为实现平台对应用的调用而设计的。需要第三方开发者显示地实现该接口，并注入到WeboSocketClient 中。
  * invokeAniObjectSync(AniStub stub)
  * sessionOnClose(String sessionId, CloseReason closeReason) session监听接口
  * sessionOnError(String sessionId, Throwable throwable) session监听接口
  
* __AgentTemplate__ 该接口是对**Service-Agent**提供的基于Http协议的业务类的单例封装模板类。利用该模板类可以方便地在应用全局以单例的方式实现对AccountService、AccountGroupService、ServiceManager、DeviceObjService和AniOAuthService的构造。

### 外部接口
* __MessageObserver__ 该接口是当对对象Stub异步调用结果的获取及处理时需要实现并注入到WebSocketClient中的。需要由第三方开发者实现。
  * void update(MessageObservable o, Object arg)

### 开发者文档
**Service-Agent**的使用都在核心类的上部利用**javadoc**进行了标注。具体的使用请参[ServiceAgent文档地址](http://bj-yatsen.anicel.cn:8080/service-agent/apidocs/)。

## 部署要求
* JDK 1.7 以上
* Hibernate Valication 包
* Spring RestTeplate 包
* anicel-commons 包

## 参考
* [Spring Document](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/)
* [Hibernate Validation](http://hibernate.org/validator/)
* [OAuth2.0 规范](https://github.com/jeansfish/RFC6749.zh-cn/blob/master/TableofContents.md)