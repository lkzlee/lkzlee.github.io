---
layout: post
title:  ONAP架构中Policy Framework详解（一）
date: 2019-11-13 11:09
categories: java技术
tags: 技术积累
description: ONAP社区看Policy相关文档自己写的总结，加深自我的理解，为国内社区对ONAP的Policy这块提供一定的技术积累和文档资料。
---
*****
* TOC
{:toc}
*****
#  ONAP架构中Policy Framework详解（一）

本文章粗粒度的介绍的ONAP Policy Framework（简称PF）框架的答题设计和所涉及的关键组件，后续我们会针对各个关键组件再次一一介绍。

## Policy Framwork是什么？

### PolicyFramework定义

Policy Framework是一个包含策略执行，部署，设计的环境，是ONAP系统架构中的决策组件。 

> The ONAP Policy Framework is a comprehensive policy design, deployment, and execution environment.  
>
> The Policy Framework is the decision making component in [an ONAP system](https://www.onap.org/wp-content/uploads/sites/20/2018/11/ONAP_CaseSolution_Architecture_112918FNL.pdf). 



Policy组件的功能。引用官方文档的一句话：`One of the most important goals of the Policy Framework is to support Policy Driven Operational Management during the execution of ONAP control loops at run time. In addition, use case implementations such as orchestration and control benefit from the ONAP policy Framework because they can use the capabilities of the framework to manage and execute their policies rather than embedding the decision making in their applications.`



说白了，Policy Framework框架就是在整个ONAP的运行时的闭环中负责执行基于策略驱动的管理操作的。它的主要特点是动态的配置，根据ONAP服务编排和控制的用例利用策略框架去管理和进行策略执行，而不是嵌入到具体ONAP服务编排的用例中去。



###  PolicyFramework简单架构

它的架构如下：

![Policy Framework架构图](/post_img/PFHighestLevel.svg)

简单来说分为四部分：Policy 开发，Policy管理，Policy执行，底层Policy DB存储。后续会详细介绍ONAP的整个框架。

### PolicyFramework详细架构：

![Policy Framework详细架构图](/post_img/PFDesignAndAdmin.svg)



**PolicyDevelopment** 负责策略元数据及策略的创建，更新，删除，查询。并且提供一套api供其他组件调用（它可以提供给其他组件诸如CLAMP进行策略的查询，创建和删除等操作）。

**PolicyAdministration**有两个重要功能：在ONAP进行安装PDPs的生命周期管理和PDPs的管理

**PolicyExecution执行**

![Policy在K8s环境中执行器架构图](/post_img/PolicyExecution.svg)

**PolicyExecution** 我称之为Policy执行器，它提供三类API：

> 增、删、改、查PDPGroup（策略组）和PDPSubGroup(策略子组)的API。
>
> PDPGroup(策略组)和PDPSubGroup(策略子组)策略控制的集合API列表。
>
> 策略执行管理的API集合、PDPGroup和SubGroup的状态、个别的PDPGroups生命周期状态API。



**名词解释**：

PAP： Policy Administration Point

PDPs：  PDPs are defined as Kubernetes Pods 

SON ：Self Organizing Network

ACPE ：Advanced Customer Premises Service

从上图中有一个重要概念，一个PDPSubGroup是同一个类型策略运行的一个PDPs组，并且每个PDPSubGroup实际的PDPs部署数量是可配置的。为了简化在K8s中的部署和水平扩展配置，以下表示官方给出的同一个策略运行在同一类型策略下的部署配置。

| PDP Group | PDP Subgroup | Kubernetes Deployment | Kubernetes Deployment Strategy                               | PDPs in Pods |
| --------- | ------------ | --------------------- | ------------------------------------------------------------ | ------------ |
| SON       | SON-XACML    | SON-XACML-Dep         | Always 2, be geo redundant                                   | 2 PDP-X      |
|           | SON-Drools   | SON-Drools-Dep        | At Least 4, scale up on 70% load, scale down on 40% load, be geo-redundant | >= 4 PDP-D   |
|           | SON-APEX     | SON-APEX-Dep          | At Least 3, scale up on 70% load, scale down on 40% load, be geo-redundant | >= 3 PDP-A   |
| ACPE      | ACPE-XACML   | ACPE-XACML-Dep        | Always 2                                                     | 2 PDP-X      |
|           | ACPE-Drools  | ACPE-Drools-Dep       | At Least 2, scale up on 80% load, scale down on 50% load     | >=2 PDP-D    |

### Policy Framework 对象模型

![Policy Framework对象关系图](/post_img/PolicyClassStructure.svg)

从图中可以看出，Policy实现类型有三种：XACML、Drool、Apex策略工具实现类（PDP Type指的也是这三类）。

### Policy Framework UML 类图

![Policy Framework类图](/post_img/PolicyDesignTimeComponents.svg)

从上图可以看到PolicyCreator--》PolicyType/Policy都是有一对一的关系，底下都有三种类型的各自实现类，详细描述了类和类之间的关系。



## Policy Design设计流程

### Policy Type Design设计开发

![策略类型设计流程图](/post_img/PolicyTypeDesign.svg)

一般Policy Framework中先需要设计开发策略类型，然后通过策略类型生成时间运行的策略，具体的一个策略类型对应于一个策略实现类（比如XACML、Drool、APEX）。

### Policy Design策略设计

![Policy Design策略设计](/post_img/PolicyDesign.svg)

策略设计一般是利用PolicyDevelopment组件从具体的Policy Type类型中创建而来。

策略创建可以通过PolicyDevelopment提供的restful **Policy Design API**来创建，也可以通过Policy GUI来创建（ONAP的portal中）。还有Policy Command Line Tool命令行工具。

### Model Driven VF（Virtual Function）Policy Design设计



![模型驱动策略设计](/post_img/ModelDrivenPolicyDesign.svg)



### Scripted Model Driven Policy Design设计

![脚本化模型驱动策略设计](/post_img/ScriptedPolicyDesign.svg)



## Policy Runtime架构设计

### Policy Framework service服务

以下服务在Policy框架中是必须的。

| Service        | Endpoint                                             | Description                                                  |
| -------------- | ---------------------------------------------------- | ------------------------------------------------------------ |
| PAP            | [https://policy-pap](https://policy-pap/)            | The PAP service, used for policy administration and deployment. See [Policy Design and Development](https://onap.readthedocs.io/en/latest/submodules/policy/parent.git/docs/design/design.html#design-label) for details of the API for this service |
| PDP-X-*domain* | [https://policy-pdpx](https://policy-pdpx/)-*domain* | A PDP service is defined for each PDP group. A PDP group is identified by the domain on which it operates.<br /><br />For example, there could be two PDP-X domains, one for admission policies for ONAP proper and another for admission policies for VNFs of operator *Supacom*. Two PDP-X services are defined:<br />[https://policy-pdpx-onap](https://policy-pdpx-onap/) |
| PDP-D-*domain* | [https://policy-pdpd](https://policy-pdpd/)-*domain* |                                                              |
| PDP-A-*domain* | [https://policy-pdpa](https://policy-pdpa/)-*domain* |                                                              |

注意：PAP服务是一个，负责策略管理和部署，和所有策略的监控；PDP服务有多个，对于每个Domain的PDP服务有很多策略。

### Policy Framework 运行时信息架构

#### 运行时相关概念

![PF运行时关系图](/post_img/RuntimeRelationships.svg)

从上图可以看到PDPSubGroup、PDPService、PolicySet、PDP、PolicyImpl间的调用关系及组件生命周期被谁所管理。

#### Policy Framework DB设计

![PF database设计](/post_img/PolicyDatabase.svg)

### Policy Framework各组件启停时序图


#### PAP启停时序图

![PAP启停](/post_img/PAPStartStop.svg)

#### PDP启停时序图

![PDP启停](/post_img/PDPStartStop.svg)

## Policy Execution执行

### Policy Execution时序图

![Policy Execution时序图](/post_img/PolicyExecutionFlow.svg)

### Policy Execution Lifecycle模式

| **Lifecycle Mode** | **Behaviour**                                                |
| ------------------ | ------------------------------------------------------------ |
| PASSIVE MODE       | Policy execution is always rejected irrespective of PDP type. |
| ACTIVE MODE        | Policy execution is executed in the live environment by the PDP. |
| SAFE MODE          | Policy execution proceeds, but changes to domain state or context are not carried out. The PDP returns an indication that it is running in SAFE mode together with the action it would have performed if it was operating in ACTIVE mode. The PDP type and the policy types it is running must support SAFE mode operation. |
| TEST MODE          | Policy execution proceeds and changes to domain and state are carried out in a test or sandbox environment. The PDP returns an indication it is running in TEST mode together with the action it has performed on the test environment. The PDP type and the policy types it is running must support TEST mode operation. |

### Policy Lifecycle management

策略生命周期管理主要指的是PDPs group在运行时管理Policy的生命周期和部署。

#### Load/Update Policy in PDP

![load/update Policy时序图](/post_img/DownloadPoliciesToPDP.svg)

#### Policy Rollout

![Policy卸载](/post_img/PolicyRollout.svg)

## 参考文档

<https://onap.readthedocs.io/en/latest/submodules/policy/parent.git/docs/architecture/architecture.html> policy framework社区文档

