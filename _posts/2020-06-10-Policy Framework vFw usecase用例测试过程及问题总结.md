---
layout: post
title:  Policy Framework vFw usecase用例测试过程及问题总结
date: 2020-06-10 14:49
categories: ONAP
tags: 技术积累
description: 本文章针对vFw usecase在onap平台测试过程中碰到的问题，进行总结，问题的解决办法，解决方案供大家参考，方便后续大家进行调试。
---

*****
* TOC
{:toc}
*****


# Policy Framework vFw usecase用例测试过程及问题总结

## vFW测试的步骤

vFW测试分为：

策略类型创建，策略组更改为ACTIVE，测略推送，策略匹配，验证AAI查询



## 策略类型创建

### 策略创建

~~~json
curl -v --silent -k --user 'healthcheck:zb!XztG34' -X POST "https://10.105.52.228:6969/policy/api/v1/policytypes" -H "Accept: application/json" -H "Content-Type: application/json" -d @policy_type.json

root@k8s-node1:~/policy-debug# cat policy_type.json
{
    "name": "ToscaServiceTemplateSimple",
    "policy_types": {
        "onap.policies.controlloop.Operational": {
            "derived_from": "tosca.policies.Root:0.0.0",
            "description": "Operational Policy for Control Loops",
            "metadata": {},
            "name": "onap.policies.controlloop.Operational",
            "properties": {},
            "version": "1.0.0"
        }
    },
    "tosca_definitions_version": "tosca_simple_yaml_1_0_0",
    "version": "1.0.0"
}
~~~



### 策略查询

~~~shell
curl -vvv --silent -k --user 'healthcheck:zb!XztG34' -X GET "https://10.105.209.215:6969/policy/pap/v1/pdps" -H "Accept: application/json" -H "Content-Type: application/json" |python -m json.tool

{
    "groups": [
        {
            "description": "The default group that registers all supported policy types and pdps.",
            "name": "defaultGroup",
            "pdpGroupState": "ACTIVE",
            "pdpSubgroups": [
                {
                    "currentInstanceCount": 0,
                    "desiredInstanceCount": 1,
                    "pdpInstances": [],
                    "pdpType": "apex",
                    "policies": [],
                    "properties": {},
                    "supportedPolicyTypes": [
                        {
                            "name": "onap.policies.controlloop.operational.Apex",
                            "version": "1.0.0"
                        }
                    ]
                },
                {
                    "currentInstanceCount": 0,
                    "desiredInstanceCount": 1,
                    "pdpInstances": [],
                    "pdpType": "drools",
                    "policies": [],
                    "properties": {},
                    "supportedPolicyTypes": [
                        {
                            "name": "onap.policies.controlloop.Operational",
                            "version": "1.0.0"
                        }
                    ]
                },
                {
                    "currentInstanceCount": 1,
                    "desiredInstanceCount": 1,
                    "pdpInstances": [
                        {
                            "healthy": "HEALTHY",
                            "instanceId": "policy-policy-xacml-pdp-77cf8dd467-2sdjq",
                            "pdpState": "ACTIVE"
                        }
                    ],
                    "pdpType": "xacml",
                    "policies": [],
                    "properties": {},
                    "supportedPolicyTypes": [
                        {
                            "name": "onap.policies.controlloop.guard.FrequencyLimiter",
                            "version": "1.0.0"
                        },
                        {
                            "name": "onap.policies.controlloop.guard.MinMax",
                            "version": "1.0.0"
                        },
                        {
                            "name": "onap.policies.controlloop.guard.Blacklist",
                            "version": "1.0.0"
                        },
                        {
                            "name": "onap.policies.controlloop.guard.coordination.FirstBlocksSecond",
                            "version": "1.0.0"
                        },
                        {
                           "name": "onap.Monitoring",
                            "version": "1.0.0"
                        },
                        {
                            "name": "onap.policies.monitoring.cdap.tca.hi.lo.app",
                            "version": "1.0.0"
                        },
                        {
                            "name": "onap.policies.monitoring.dcaegen2.collectors.datafile.datafile-app-server",
                            "version": "1.0.0"
                        },
                        {
                            "name": "onap.policies.monitoring.docker.sonhandler.app",
                            "version": "1.0.0"
                        },
                        {
                            "name": "onap.policies.optimization.AffinityPolicy",
                            "version": "1.0.0"
                        },
                        {
                            "name": "onap.policies.optimization.DistancePolicy",
                            "version": "1.0.0"
                        },
                        {
                            "name": "onap.policies.optimization.HpaPolicy",
                            "version": "1.0.0"
                        },
                        {
                            "name": "onap.policies.optimization.OptimizationPolicy",
                            "version": "1.0.0"
                        },
                        {
                            "name": "onap.policies.optimization.PciPolicy",
                            "version": "1.0.0"
                        },
                        {
                            "name": "onap.policies.optimization.QueryPolicy",
                            "version": "1.0.0"
                        },
                        {
                            "name": "onap.policies.optimization.SubscriberPolicy",
                           "version": "1.0.0"
                        },
                        {
                            "name": "onap.policies.optimization.Vim_fit",
                            "version": "1.0.0"
                        },
                        {
                            "name": "onap.policies.optimization.VnfPolicy",
                            "version": "1.0.0"
                        }
                    ]
                }
            ],
            "properties": {}
        }
    ]
}
~~~



## 策略创建

~~~shell
curl -v --silent -k --user 'healthcheck:zb!XztG34' -X POST "https://10.105.52.228:6969/policy/api/v1/policytypes/onap.policies.controlloop.Operational/versions/1.0.0/policies" -H "Accept: application/json" -H "Content-Type: application/json" -d @vfwcl.policy.operational.input.json


{
  "policy-id" : "operational.modifyconfig",
  "content" : "controlLoop%3A%0A++++version%3A+2.0.0%0A++++controlLoopName%3A+ControlLoop-vFirewall-d0a1dfc6-94f5-4fd4-a5b5-4630b438850a%0A++++trigger_policy%3A+unique-policy-id-1-modifyConfig%0A++++timeout%3A+1200%0A++++abatement%3A+false%0Apolicies%3A%0A++++-+id%3A+unique-policy-id-1-modifyConfig%0A++++++name%3A+modify_packet_gen_config%0A++++++description%3A%0A++++++actor%3A+APPC%0A++++++recipe%3A+ModifyConfig%0A++++++target%3A%0A++++++++++resourceID%3A+97ab88b8-afaa-4eaa-b575-4a29f7f8c0bb%0A++++++++++type%3A+VNF%0A++++++payload%3A%0A++++++++++streams%3A+%27%7B%22active-streams%22%3A5%7D%27%0A++++++retry%3A+0%0A++++++timeout%3A+300%0A++++++success%3A+final_success%0A++++++failure%3A+final_failure%0A++++++failure_timeout%3A+final_failure_timeout%0A++++++failure_retries%3A+final_failure_retries%0A++++++failure_exception%3A+final_failure_exception%0A++++++failure_guard%3A+final_failure_guard%0A" 
}
~~~



## 部署Policy

~~~shell
curl -vvv --silent -k --user 'healthcheck:zb!XztG34' -X POST "https://10.105.209.215:6969/policy/pap/v1/pdps/policies" -H "Accept: application/json" -H "Content-Type: application/json" -d @vfwcl.push.json

vi vfwcl.push.json
{
  "policies": [
    {
      "policy-id": "operational.modifyconfig",
      "policy-version": "1.0.0"
    }
  ]
}
~~~



## AAI验证查询

Policy会调用下面这个AAI api，获取vserver的resoucelink，然后再调用对应的连接获取执行结果。

查询AAI调用的resource-link：

~~~shell
curl --location --request GET 'https://aai.api.sparky.simpledemo.onap.org:30233/aai/v16/search/nodes-query?search-node-type=vserver&filter=vserver-name:EQUALS:zdfw1fwl01fwl01' \
--header 'Content-Type: application/json' \
--header 'Authorization: Basic QUFJOkFBSQ==' \
--header 'X-FromAppId: jimmy-postman' \
--header 'Accept: application/json' \
--header 'X-TransactionId: 9999'

返回如下：{
    "result-data": [
        {
            "resource-type": "vserver",
            "resource-link": "/aai/v16/cloud-infrastructure/cloud-regions/cloud-region/CMCC/RegionOne/tenants/tenant/c39bad9117904e71b523c5f9ed012af4/vservers/vserver/8e15d696-c36b-4512-9387-741dc02f552a"
        }
    ]
}
~~~



# 碰到的问题



## 证书过期问题

Exception 信息：

~~~java
[2020-05-07T01:16:00.547+00:00|ERROR|InlineBusTopicSink|DMAAP-source-APPC-LCM-WRITE] SingleThreadedDmaapTopicSource [userName=null, password=-, getTopicCommInfrastructure()=DMAAP, toString()=SingleThreadedBusTopicSource [consumerGroup=05338d21-5e50-46b0-87d3-9ba32609d467, consumerInstance=policy-drools-0, fetchTimeout=15000, fetchLimit=100, consumer=CambriaConsumerWrapper [fetchTimeout=15000], alive=true, locked=false, uebThread=Thread[DMAAP-source-APPC-LCM-WRITE,5,main], topicListeners=2, toString()=BusTopicBase [apiKey=null, apiSecret=null, useHttps=true, allowSelfSignedCerts=false, toString()=TopicBase [servers=[message-router], topic=APPC-LCM-WRITE, effectiveTopic=APPC-LCM-WRITE, #recentEvents=0, locked=false, #topicListeners=2]]]]: cannot fetch because of
javax.net.ssl.SSLHandshakeException: sun.security.validator.ValidatorException: PKIX path validation failed: java.security.cert.CertPathValidatorException: validity check failed
        at sun.security.ssl.Alerts.getSSLException(Alerts.java:192)
        at sun.security.ssl.SSLSocketImpl.fatal(SSLSocketImpl.java:1946)
        at sun.security.ssl.Handshaker.fatalSE(Handshaker.java:316)
        at sun.security.ssl.Handshaker.fatalSE(Handshaker.java:310)
        at sun.security.ssl.ClientHandshaker.serverCertificate(ClientHandshaker.java:1639)
        at sun.security.ssl.ClientHandshaker.processMessage(ClientHandshaker.java:223)
        at sun.security.ssl.Handshaker.processLoop(Handshaker.java:1037)
        at sun.security.ssl.Handshaker.process_record(Handshaker.java:965)
        at sun.security.ssl.SSLSocketImpl.readRecord(SSLSocketImpl.java:1064)
        at sun.security.ssl.SSLSocketImpl.performInitialHandshake(SSLSocketImpl.java:1367)
        at sun.security.ssl.SSLSocketImpl.startHandshake(SSLSocketImpl.java:1395)
        at sun.security.ssl.SSLSocketImpl.startHandshake(SSLSocketImpl.java:1379)
        at org.apache.http.conn.ssl.SSLConnectionSocketFactory.createLayeredSocket(SSLConnectionSocketFactory.java:436)
        at org.apache.http.conn.ssl.SSLConnectionSocketFactory.connectSocket(SSLConnectionSocketFactory.java:384)
        at org.apache.http.impl.conn.DefaultHttpClientConnectionOperator.connect(DefaultHttpClientConnectionOperator.java:142)
        at org.apache.http.impl.conn.PoolingHttpClientConnectionManager.connect(PoolingHttpClientConnectionManager.java:374)
        at org.apache.http.impl.execchain.MainClientExec.establishRoute(MainClientExec.java:393)
        at org.apache.http.impl.execchain.MainClientExec.execute(MainClientExec.java:236)
        at org.apache.http.impl.execchain.ProtocolExec.execute(ProtocolExec.java:186)
        at org.apache.http.impl.execchain.RetryExec.execute(RetryExec.java:89)
        at org.apache.http.impl.execchain.RedirectExec.execute(RedirectExec.java:110)
        at org.apache.http.impl.client.InternalHttpClient.doExecute(InternalHttpClient.java:185)
        at org.apache.http.impl.client.CloseableHttpClient.execute(CloseableHttpClient.java:83)
        at com.att.nsa.apiClient.http.HttpClient.runCall(HttpClient.java:708)
        at com.att.nsa.apiClient.http.HttpClient.get(HttpClient.java:384)
        at com.att.nsa.apiClient.http.HttpClient.get(HttpClient.java:368)
        at com.att.nsa.cambria.client.impl.CambriaConsumerImpl.fetch(CambriaConsumerImpl.java:87)
        at com.att.nsa.cambria.client.impl.CambriaConsumerImpl.fetch(CambriaConsumerImpl.java:64)
        at org.onap.policy.common.endpoints.event.comm.bus.internal.BusConsumer$CambriaConsumerWrapper.fetch(BusConsumer.java:171)
        at org.onap.policy.common.endpoints.event.comm.bus.internal.SingleThreadedBusTopicSource.fetchAllMessages(SingleThreadedBusTopicSource.java:238)
        at org.onap.policy.common.endpoints.event.comm.bus.internal.SingleThreadedBusTopicSource.run(SingleThreadedBusTopicSource.java:228)
        at java.lang.Thread.run(Thread.java:748)
Caused by: sun.security.validator.ValidatorException: PKIX path validation failed: java.security.cert.CertPathValidatorException: validity check failed
        at sun.security.validator.PKIXValidator.doValidate(PKIXValidator.java:362)
        at sun.security.validator.PKIXValidator.engineValidate(PKIXValidator.java:270)
        at sun.security.validator.Validator.validate(Validator.java:262)
        at sun.security.ssl.X509TrustManagerImpl.validate(X509TrustManagerImpl.java:324)
        at sun.security.ssl.X509TrustManagerImpl.checkTrusted(X509TrustManagerImpl.java:229)
        at sun.security.ssl.X509TrustManagerImpl.checkServerTrusted(X509TrustManagerImpl.java:124)
        at sun.security.ssl.ClientHandshaker.serverCertificate(ClientHandshaker.java:1621)
        ... 27 common frames omitted
Caused by: java.security.cert.CertPathValidatorException: validity check failed
        at sun.security.provider.certpath.PKIXMasterCertPathValidator.validate(PKIXMasterCertPathValidator.java:135)
        at sun.security.provider.certpath.PKIXCertPathValidator.validate(PKIXCertPathValidator.java:233)
        at sun.security.provider.certpath.PKIXCertPathValidator.validate(PKIXCertPathValidator.java:141)
        at sun.security.provider.certpath.PKIXCertPathValidator.engineValidate(PKIXCertPathValidator.java:80)
        at java.security.cert.CertPathValidator.validate(CertPathValidator.java:292)
        at sun.security.validator.PKIXValidator.doValidate(PKIXValidator.java:357)
        ... 33 common frames omitted
~~~

解决方式：

替换证书，社区给出的解决方式，替换这些配置重新打包，部署即可解决：

 https://gerrit.onap.org/r/c/oom/+/107265
https://gerrit.onap.org/r/c/oom/+/105913 



## Policy DefaultGroup State问题

在push策略是有以下报错，最后发现是DefaultGroup State状态不对，对State状态进行修改即可。

执行命令：

>  curl -vvv --silent -k --user 'healthcheck:zb!XztG34' -X POST "https://10.105.209.215:6969/policy/pap/v1/pdps/policies" -H "Accept: application/json" -H "Content-Type: application/json" -d @vfwcl.push.json

~~~shell
vi vfwcl.push.json

{
 "policies": [
  {
   "policy-id": "operational.modifyconfig",
   "policy-version": "1.0.0"
  }
 ]
}

Response Error like:
{
  "errorDetails": "policy not supported by any PDP group: operational.modifyconfig 1.0.0"
}
~~~

查询了下PDP组的状态是PASSIVE ，按照官方文档他会拒绝一切执行的策略。

~~~shell
curl -vvv --silent -k --user 'healthcheck:zb!XztG34' -X GET "https://10.105.209.215:6969/policy/pap/v1/pdps" -H "Accept: application/json" -H "Content-Type: application/json"
{
    "groups": [
        {
            "description": "The default group that registers all supported policy types and pdps.",
            "name": "defaultGroup",
            "pdpGroupState": "PASSIVE",
            "pdpSubgroups": [
                {
                    "currentInstanceCount": 0,
                    "desiredInstanceCount": 1,
                    "pdpInstances": [],
                    "pdpType": "apex",
                    "policies": [],
                    "properties": {},
                    "supportedPolicyTypes": [
                        {
                            "name": "onap.policies.controlloop.operational.Apex",
                            "version": "1.0.0"
                        }
                    ]
                },
                {
                    "currentInstanceCount": 0,
                    "desiredInstanceCount": 1,
                    "pdpInstances": [],
                    "pdpType": "drools",
                    "policies": [],
                    "properties": {},
                    "supportedPolicyTypes": [
                        {
                            "name": "onap.policies.controlloop.Operational",
                            "version": "1.0.0"
                        }
                    ]
                },
                {
                    "currentInstanceCount": 0,
                    "desiredInstanceCount": 1,
                    "pdpInstances": [],
                    "pdpType": "xacml",
                    "policies": [],
                    "properties": {},
                    "supportedPolicyTypes": [
                        {
                            "name": "onap.policies.controlloop.guard.FrequencyLimiter",
                            "version": "1.0.0"
                        },
                     	...
                    ]
                }
            ],
            "properties": {}
        }
    ]
}
~~~

更改PASSIVE为ACTIVE状态：

~~~shell
curl -vvv --silent -k --user 'healthcheck:zb!XztG34' -X PUT "https://10.105.209.215:6969/policy/pap/v1/pdps/groups/defaultGroup?state=ACTIVE" -H "Accept: application/json" -H "Content-Type: application/json"
~~~



## Push Policy测试失败问题



##  策略不匹配

从network.log DCAE收到的实际发现，closedLoopControlName 和配置的Policy不一致，导致不能匹配上。

~~~shell
{"closedLoopEventClient":"DCAE_INSTANCE_ID.dcae-tca","policyVersion":"v0.0.1","policyName":"DCAE.Config_tca-hi-lo","policyScope":"DCAE","target_type":"VM","AAI":{"vserver.vserver-name":"zdfw1fwl01fwl01"},"closedLoopAlarmStart":1588926185199739,"closedLoopEventStatus":"ONSET","closedLoopControlName":"ControlLoop-vFirewall-d0a1dfc6-94f5-4fd4-a5b5-4630b438850a","version":"1.0.2","target":"vserver.vserver-name","requestID":"c1e58beb-a2b0-45bf-b69d-758b91f91a79","from":"DCAE"}
~~~

从telemetry查询看到：

~~~shell
{"closedLoopControlName": "ControlLoop-vFirewall-135835e3-eed7-497a-83ab-8c315f37fa4a",...} 

https://localhost:9696/policy/pdp/engine/controllers/usecases/drools/facts/useca

ses> get

HTTP/1.1 200 OK

Content-Length: 124

Content-Type: application/json

Date: Fri, 08 May 2020 09:02:25 GMT

Server: Jetty(9.4.20.v20190813)

{
    "org.onap.policy.controlloop.params.ControlLoopParams": 1, 
    "org.onap.policy.models.tosca.authorative.concepts.ToscaPolicy": 1
}

https://localhost:9696/policy/pdp/engine/controllers/usecases/drools/facts/useca
ses/org.onap.policy.controlloop.params.ControlLoopParams> get
HTTP/1.1 200 OK
Content-Length: 1125
Content-Type: application/json
Date: Fri, 08 May 2020 09:02:39 GMT
Server: Jetty(9.4.20.v20190813)
[
    {
        "closedLoopControlName": "ControlLoop-vFirewall-135835e3-eed7-497a-83ab-8c315f37fa4a", 
        "controlLoopYaml": "controlLoop%3A%0A++++version%3A+2.0.0%0A++++controlLoopName%3A+ControlLoop-vFirewall-135835e3-eed7-497a-83ab-8c315f37fa4a%0A++++trigger_policy%3A+unique-policy-id-1-modifyConfig%0A++++timeout%3A+1200%0A++++abatement%3A+false%0Apolicies%3A%0A++++-+id%3A+unique-policy-id-1-modifyConfig%0A++++++name%3A+modify_packet_gen_config%0A++++++description%3A%0A++++++actor%3A+APPC%0A++++++recipe%3A+ModifyConfig%0A++++++target%3A%0A++++++++++resourceID%3A+97ab88b8-afaa-4eaa-b575-4a29f7f8c0bb%0A++++++++++type%3A+VNF%0A++++++payload%3A%0A++++++++++streams%3A+%27%7B%22active-streams%22%3A5%7D%27%0A++++++retry%3A+0%0A++++++timeout%3A+300%0A++++++success%3A+final_success%0A++++++failure%3A+final_failure%0A++++++failure_timeout%3A+final_failure_timeout%0A++++++failure_retries%3A+final_failure_retries%0A++++++failure_exception%3A+final_failure_exception%0A++++++failure_guard%3A+final_failure_guard%0A", 
        "policyName": "operational.modifyconfig", 
        "policyScope": "onap.policies.controlloop.Operational:1.0.0", 
        "policyVersion": "1.0.0"
    }
]
https://localhost:9696/policy/pdp/engine/controllers/usecases/drools/facts/useca
ses/org.onap.policy.models.tosca.authorative.concepts.ToscaPolicy> get
HTTP/1.1 200 OK
Content-Length: 1269
Content-Type: application/json
Date: Fri, 08 May 2020 09:03:44 GMT
Server: Jetty(9.4.20.v20190813)
[
    {
        "identifier": {
            "name": "operational.modifyconfig", 
            "version": "1.0.0"
        }, 
        "key": {
            "name": "operational.modifyconfig", 
            "version": "1.0.0"
        }, 
        "metadata": {}, 
        "name": "operational.modifyconfig", 
        "properties": {
            "content": "controlLoop%3A%0A++++version%3A+2.0.0%0A++++controlLoopName%3A+ControlLoop-vFirewall-135835e3-eed7-497a-83ab-8c315f37fa4a%0A++++trigger_policy%3A+unique-policy-id-1-modifyConfig%0A++++timeout%3A+1200%0A++++abatement%3A+false%0Apolicies%3A%0A++++-+id%3A+unique-policy-id-1-modifyConfig%0A++++++name%3A+modify_packet_gen_config%0A++++++description%3A%0A++++++actor%3A+APPC%0A++++++recipe%3A+ModifyConfig%0A++++++target%3A%0A++++++++++resourceID%3A+97ab88b8-afaa-4eaa-b575-4a29f7f8c0bb%0A++++++++++type%3A+VNF%0A++++++payload%3A%0A++++++++++streams%3A+%27%7B%22active-streams%22%3A5%7D%27%0A++++++retry%3A+0%0A++++++timeout%3A+300%0A++++++success%3A+final_success%0A++++++failure%3A+finalure_guard%3A+final_failure_guard%0A"
        }, 
        "type": "onap.policies.controlloop.Operational", 
        "typeIdentifier": {
            "name": "onap.policies.controlloop.Operational", 
            "version": "1.0.0"
        }, 
        "typeVersion": "1.0.0", 
        "version": "1.0.0"
    }
]
~~~



但是此时Policy查询的结果 closedLoopControlName 不一致，导致后续Policy策略不能匹配触发。

## AAI为空，插入AAI数据



插入AAI数据：

~~~shell
curl --location --request PUT 'https://aai.api.sparky.simpledemo.onap.org:30233/aai/v16/cloud-infrastructure/cloud-regions/cloud-region/CMCC/RegionOne/tenants/tenant/c39bad9117904e71b523c5f9ed012af4/vservers/vserver/8e15d696-c36b-4512-9387-741dc02f552a' \
--header 'Accept: application/json' \
--header 'X-FromAppId: jimmy-postman' \
--header 'X-TransactionId: 9999' \
--header 'Authorization: Basic QUFJOkFBSQ==' \
--header 'Content-Type: text/plain' \
--data-raw '{
  "vserver-id": "8e15d696-c36b-4512-9387-741dc02f552a",
  "vserver-name": "zdfw1fwl01fwl01",
  "prov-status": "ACTIVE",
  "vserver-selflink": "/aai/v16/cloud-infrastructure/cloud-regions/cloud-region/CMCC/RegionOne/tenants/tenant/c39bad9117904e71b523c5f9ed012af4/vservers/vserver/8e15d696-c36b-4512-9387-741dc02f552a",
  "in-maint": true,
  "is-closed-loop-disabled": false,
  "resource-version": "1589274039713",
  "relationship-list": [
    {
      "relationship": [
        {
          "related-to": "generic-vnf",
          "related-link": "/aai/v16/network/generic-vnfs/generic-vnf/9d8893f4-f158-451a-9828-46dfc610de71"
        }
      ]
    }
  ],
  "admin-status": "string"
}
  '
~~~



# 参考资料





