---
layout: post
title:  Policy Framework策略笔记总结
date: 2020-03-13 11:09
categories: ONAP
tags: 技术积累
description: ONAP社区看Policy相关文档自己写的总结，主要总结下PF中的策略类型。

---

*****
* TOC
{:toc}
*****

# Policy Framework策略笔记总结

## Policy Elato版本组件总结

1、对比之前ZTE 文档，在onap社区Policy部分着重学习Policy预制策略和策略类型内容，并总结

2、和之前阅读的中兴Policy梳理材料对比，进行策略总结。

### Policy策略类型总结

策略的种类按部署分为两种：全局策略、局部策略。

全局策略——与具体业务无关，可独立部署

局部策略——与具体业务相关，与服务一起部署，如NS scaling

按业务分为：(ETSI用例)

> VNF LCM中的授权策略
>
> NFVI-PoPs中的资源分配策略
>
> NFVO中的NS healing策略
>
> VNFD中预定义的自动缩、扩容策略
>
> VNF LCM与VNF实例关联策略
>
> EM创建的VNF healing策略

按策略执行点分：

若NFVO为策略执行点：

> 策略包括NS instantiation policy, NS scaling policy, NS updating policy, NS healing policy,  NS termination policy.

若VNFM为策略执行点：

> 策略包括VNF instantiation policy, VNF scaling policy, VNF healing policy, VNF termination policy.

### elalto版本Policy组件：

| Policy Pod          | Latest Framework | Legacy |
| ------------------- | ---------------- | ------ |
| brmsgw              |                  | yes    |
| drools              | yes              | yes    |
| nexus               | yes              | yes    |
| pap                 |                  | yes    |
| pdp                 |                  | yes    |
| policy-apex-pdp     | yes              |        |
| policy-api          | yes              |        |
| policy-distribution | yes              | yes    |
| policy-pap          | yes              |        |
| policy-xacml-pdp    | yes              |        |
| policydb            | yes              | yes    |

## vFW在Policy组件处理测试流程

### vfw流程在Policy模块的功能点？

vFW流以DCAE发送的起始消息开始，该消息通知PDP-D需要对VNF采取操作。一旦PDP-D将起始点插入drools内存，规则就开始启动，以开始处理drools内存中存在的vFW策略的起始点。如果起始点没有完整的的A&AI数据，策略将查询VNF数据的A&AI，否则PDP-D将从起始点直接获得所需的A&AI数据。接着从一开始就对源VNF实体执行一个名为A&AI的查询，以查找PDP-D将要对其执行操作的目标VNF实体。从A&AI中检索到目标实体后，将执行一个保护查询，以确定是否允许执行要执行的操作。如果Guard返回许可证，PDP-D将发送APPC ModifyConfig recipe请求以修改请求负载中指定的pg流。如果APPC成功，那么PDP-D将发送关于POLICY-CL-MGT主题的最终成功通知，并优雅地结束对事件的处理。

### Policy Drools Features关联组件打开设置

vFW初始设置：

首先关闭policy，`policy stop`

然后Policy 的Drools引擎打开对应的组件，执行命令` features enable controlloop-utils` 

最后，启动Drools组件，`policy start`



### Policy中vFW对应的API调用

1. 查看内存中 Telemetry  Param对象，应该包含vFW策略：

> ```
> curl -k --silent --user @1b3rt:31nst31n -X GET https://10.107.43.80:9696/policy/pdp/engine/controllers/amsterdam/drools/facts/amsterdam | python -m json.tool
> ```

2. 使用API写入一个DACE事件：

> ```
> curl -k --silent --user @1b3rt:31nst31n --header "Content-Type: text/plain" --data @dcae.vfw.onset.json -X PUT https://10.107.43.80:9696/policy/pdp/engine/topics/sources/ueb/unauthenticated.DCAE_EVENT_OUTPUT/events | python -m json.tool
> ```

3. 现在查看一下 Telemetry  内存中的Param对象，应该有7个。

PDP-D组件的日志：

日志位于$POLICY_HOME/logs/network.log。可以查看PDP-D在处理的不同阶段发出的通知。日志成功处理的顺序以活动通知开始，以及表明已开始确认，及相关操作正在处理等。



4. 接下来PDP-D会向A&AAI查询，以后去从开始指定的VNF信息，下图展示了这个流程（日志截图）：

![Tut_vFW_aai_get.jfif](/post_img/2020/Tut_vFW_aai_get.jfif)



5. 对于vFW用例，起始消息中报告的源实体可能不是APPC操作对其执行操作的目标实体。为了确定真正的目标实体，将执行一个名为A&AI的查询。请求显示在network.log日志中。

![Tut_vFW_aai_named_query_request.jfif](/post_img/2020/Tut_vFW_aai_named_query_request.jfif)

返回结果也会打印到日志中，如图：

![Tut_vFW_aai_named_query_response.jfif](/post_img/2020/Tut_vFW_aai_named_query_response.jfif)

6.一旦找到目标实体，PDP-D最终会确定是否应允许此操作，将发送一系列操作通知以启动Guard查询、获取许可或拒绝并开始操作。

开始查询操作

![Tut_vFW_policy_guard_start.jfif](/post_img/2020/Tut_vFW_policy_guard_start.jfif)

查询结果



![Tut_vFW_policy_guard_result.jfif](/post_img/2020/Tut_vFW_policy_guard_result.jfif)

操作开始

![Tut_vFW_policy_operation_start.jfif](/post_img/2020/Tut_vFW_policy_operation_start.jfif)



7. 在操作开始时，会发送一个appc操作，如下图：

![2020/Tut_vFW_appc_request.jfif](/post_img/Tut_vFW_appc_request.jfif)

8.APPC的请求使用APPC-CL topic事件作为 响应结果（这个是模拟结果，下面告诉模拟方法）

![Tut_vFW_simulated_appc_response.jfif](/post_img/2020/Tut_vFW_simulated_appc_response.jfif)

上面结果是模拟的事件，发送下面api即可模拟这个响应请求：

```
curl -k --silent --user @1b3rt:31nst31n --header "Content-Type: text/plain" --data @appc.legacy.success.json -X PUT https://localhost:9696/policy/pdp/engine/topics/sources/ueb/APPC-CL/events | python -m json.tool
```

9. 接下来network.log不出意外的话会显示操作成功的日志。

   

![Tut_vFW_policy_operation_success.jfif](/post_img/2020/Tut_vFW_policy_operation_success.jfif)

10. 最终vFW测试在Policy决策验证结束。处理完成后Telemetry  Param对象又只剩一个，说明策略可重入的，之前的vFw准备工作被清除，可以重新开始操作测试。

    

