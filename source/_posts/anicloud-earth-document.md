---
title: Ani.Cloud Earth Document
date: 2016-10-18 13:59:15
categories: Ani.Cloud
tags: account
top: true
---

## 概述
**Earth**模块是提供整个Anicloud 平台的基础用户账号管理模块。该模块定义了系统用户的基本信息、用户类别，
同时该模块还定义了用户组等概念。为了实现对OAuth2.0 的支持，系统还扩展了对OAuth2.0必备的信息的存储。通过用户组的
概念定义了用户资源的访问权限。该模块还提供了用户联系人的概念，并在底层给予支持。系统对用户的密码采用**BCrypt**
加密算法进行加密，该问题在集成**Ani-CAS**系统时需要考虑。
<!--more -->
## 功能列表
### 用户用户组
> 用户用户组提供了对用户的基本管理功能。系统将用户分为个人用户、企业用户和ROOT用户。ROOT用户为系统的超级用户，
负责创建系统用户组。系统默认会将每一个注册的用户添加到默认的用户组中。

### 用户联系人列表
> 用户联系人提供用户对自己的联系人进行管理的功能。用户的联系人列表是用户对跟自己经常产生关系的其他用户的集合。

## 设计实现
### 技术选择
* Spring framework
* Spring framework Security
* Hibernate JPA
* Redis Cache
* BCrypt 加密
* MySQL

### 系统设计
#### 模块划分
* __Account-Manager模块__ 该模块包括系统的核心业务，数据持久化等。
* __Ostopus-Service模块__ 提供Account-Manager模块的发布部署操作，包括对外基于Http的接口发布。基于RMI的业务接口发布等。

#### 设计
该模块采用基于DDD的领域驱动设计。由Spring的Configuralbe接口提供技术支持。

### 核心业务类
* __AccountService__ 提供对用户的基本操作。包括为用户关联用户组。
* __AccountGroupService__ 提供对用户组的基本操作。
* __AccountContantService__ 提供对用户联系人列表的基本操作。
* __GroupJoinInvitationService__ 提供对用户关联用户组的消息管理功能。

### 对外接口
* __AccountServiceFacade__ 提供对用户管理的门面封装，最终通过RMI方式对外暴漏。
* __AccountGroupServiceFacade__ 提供对用户组管理的门面封装，最终通过RMI方式对外暴漏。
* __AccountContactServiceFacade__ 提供对用户联系人列表的操作功能的封装。
* __GroupJoinInvitationServiceFacade__ 提供对关联用户组操作消息的封装。    
* __CachedAccountServiceFacadeImpl__ 提供对用户基本信息读取的基于Redis的缓存封装，采用装饰者模式实现。

### 外部接口
**Earth**模块对外暴漏的接口，其他子系统都将通过**Octopus-Antenna**模块进行访问。

## 部署要求
部署要求参见**Service-Bus**的部署要求。[ServiceBus部署要求](https://github.com/anicloud/anibus/blob/master/service-bus-master/doc/Service%20Bus%20Document.md#部署要求)。

## 引用
* [Spring Document](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/)
* [Spring Security](http://projects.spring.io/spring-security/)
* [OAuth2.0 规范](https://github.com/jeansfish/RFC6749.zh-cn/blob/master/TableofContents.md)
