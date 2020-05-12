---
layout: post
title:  Policy Framework软件架构（二）
date: 2020-01-03 11:09
categories: ONAP
tags: 技术积累
description: ONAP社区看Policy相关文档自己写的总结，Policy Framework以下简称为PF，以下文章主要讲解PF的软件架构。

---

*****
* TOC
{:toc}
*****


# Policy Framework软件架构（二）

Policy Framework以下简称为PF。

# PF框架的软件架构



PF的 从软件角度出发，架构模块主要包含以下：

> PDP-D 软件模块架构
>
> PDP-X 软件模块架构
>
> PAP 软件模块架构
>
> BRMSGW 软件模块架构



# PDP-D软件架构

## 模块划分：

> policy-utils (工具库)
>
> policy-core （drool库接口）
>
> policy-endpoints （网络）
>
> policy-management （平台管理）

这些组件组成了最小的服务集合来提供Drools-Application的执行。

PDP-D服务是一个轻量级的基于Drools引擎的Java Application应用提供通用服务，可以做水平横向扩展（0..n)。

环境要求：JDK8

## PDP-D的特性：

主要包含以下特性：

>Feature Test Transaction (disabled by default)
>Feature State Management (disabled by default)
>Feature EELF (disabled by default)
>Feature Healthcheck (enabled by default)
>Feature Session Persistence (disabled by default)
>Feature Active/Standby Management (disabled by default)
>Feature Distributed Locking (enabled in OOM installation by default)
>Feature Pooling (enabled in OOM installation by default)
>Feature Controller Logging (disabled by default)
>Feature MDC Filters (disabled by default)





# PDP-X软件架构

PDP-X主要接收XACML请求和响应。

它主要分为两个项目：

> ONAP-PDP (core PDP decision component)
>
> ONAP-PDP-REST (restfult 接口封装的服务，用于PAP通信及处理PDP客户端的请求)

PDP-X 其中的 **ONAP-PDP-REST**  主要承担处理来自PAP的请求和遗留的XACML请求的代理api，它是一个包含包装器代码和宿主策略api的服务，还会充当策略管理相关api的代理。

ONAP-PDP项目是PDP的XACML实现的一个扩展，它包含用于策略决策过程的核心XACML功能和定义。

环境要求：

> JDK1.8
>
> Tomcat8作为PDP-X的web容器
>
> 文件和Properties文件用户存储策略信息，用于PDP发生故障时恢复
>
> 内存缓存用于缓存策略信息，这样可以在启动后，快速处理服务请求。

​				

# PAP软件架构

PAP用于管理Drools和XACML策略组件，它本身底层核心基于XACML实现，组件本身属于XACML家族体系，这个组件的主要作用是管理PDP配置，并负责策略的创建、删除、更新操作。

## PAP实现的主要功能：

> 管理组，创建、删除或编辑组
>
> 管理PDP组件，
>
> 管理策略，创建，更新和删除数据库中的策略。将策略分配给某个组，也可以从组中添加或删除。
>
> 管理组的PIP配置，PIP（策略信息点，Policy Information Point）。
>
> 提供RESTful api接口，供PDP与PAP通信，以便注册和更新它们的属性。
>
> 提供RESTful api接口，供PDP/Policy-SDK-APP（Policy GUI）调用PAP并利用其服务。

  						

PAP的主要部署项目为：

>  ONAP-PAP-REST → https://git.onap.org/policy/engine/tree/ONAP-PAP-REST 

环境要求：

> JDK1.8
>
> Tomcat8作为PDP-X的web容器
>
>  MariaDB 用于存储Policy策略信息
>
> 文件和Properties文件用于存储组信息
>
> ElasticSearch存储用于索引和检索的策略数据。

其中PAP中有一个特殊的PDP更新线程用于在组发生任何变更时更新所有PDP。

# BRMSGW 软件架构

BRMS是业务规则管理系统的定义。GW的意思为网关，Gateway。

BRMSGW充当PDP-D和PDP-X之间的接口，以便Drools规则管理与分配PDP-D，它管理PDP-D中的规则，并且是闭环体系结构中的一部分。

BRMSGW的主要功能：

> PDP-D需要nexus中的规则artifacts来获取它们，因此BRMSGW将BRMS配置规则从PDP-X.drl格式转换为PDP-D的nexus格式。
>
> BRMSGW监听来自基于BRMS策略的任何消息通知
>
> 它管理者PDP-D的artifacts和jar的控制器和依赖项，一旦受到通知，BRMS规则（.drl文件）会将从PDP-X中提取并更新到maven项目中，执行maven deploy以创建规则jar，并push到nexus，使其PDP-D可用。
>
> DMaaP/UEB通知将被发送到有关新规则artiface的PDP-D



环境要求：

> jdk8
>
> 独立的java application，支持运行多个BRMSGW实例。
>
> MariaDB 为了保持和其他Controller同步，规则信息被用了存储在数据库中做同步。
>
> 文件-属性文件、配置文件和maven规则存储在文件中。
>
> Maven- 依赖于maven installation，因此需要Maven和Maven的settings.xml文件才能成功运行。

# XACML 外编

（XACML）可扩展的访问控制标记语言是由[OASIS组织](https://baike.baidu.com/item/OASIS组织)开发，采用[XML](https://baike.baidu.com/item/XML)表示的访问控制策略语言。

XACML策略语言允许管理员定义访问控制需求。XACML还包括一种访问决策语言，用于描述对资源运行时的请求。当确定了保护资源的策略之后，函数会将请求中的属性与包含在策略规则中的属性进行比较，最终生成一个许可或拒绝决策。

简言之，XACML是一种新的用于管理策略和访问控制的标记语言，同时又是一种通用的访问控制策略定义语言，提供一整套语法(使用XML定义)来管理对系统资源的访问。

目前，多数系统都以专有的方式实现访问控制和授权，在专有访问控制系统中，实体及其属性的信息保存在资料库，即访问控制列表中。不同的专有系统具有不同的实现[ACL](https://baike.baidu.com/item/ACL)的机制，因此难以交换和共享信息。同时，这些机制缺少表示复杂策略(在现实系统中经常需要用到)的能力。因此，访问控制策略通常会嵌入应用程序代码中，这使得更改策略(或者只是找出哪些策略正在实施)变得很困难。

XACML的出现使得不同环境中可以简单、灵活地制定各种访问控制策略。XACML的通用性使得各系统之间的访问控制策略和过程得到标准化。XACML是一种主要由机器生成的语言，它们能用于多个应用程序，并可以实现不同系统之间访问控制的互操作。



XACML有如下优点：

- 1、安全管理员只需对访问控制策略描述一次，不必在不同的系统中使用不同的应用程序策略语言重写多次。

- 2、应用程序开发者不必开发自己的策略语言和编写支持它们的程序，他们可以重复使用已有的和标准化的程序。

- 3、XACML能适应大多数访问控制策略的需求，当新的访问控制要求出现时，只需要加入策略，不必修改应用程序。

- 4、单一XACML策略能应用于多个资源，这有助于在为不同资源编制策略时，避免不一致性和重复劳动。

- 5、XACML中一个策略可以引用另一个策略。

# 总结

Drools PDP主要实现支持对组件（如SO/APPC/VFC/SDNC/SDNR）执行的运行时控制循环操作。

XACML PDP主要在DCAE策略处理程序运行时支持问题/答案策略决策。

PAP主要用户策略管理，组管理，提供RESTful api接口和PDP进行通信，保证信息更新和同步。

BRMSGW用于PDP-D和PDP-X之间的接口，以便Drools规则可以被管理与分配到PDP-D。

# 参考文档

 https://onap.readthedocs.io/en/latest/submodules/policy/engine.git/docs/platform/index.html  PF框架旧的系统的软件模块

