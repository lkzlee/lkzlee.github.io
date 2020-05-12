
---
layout: post
title:  Policy Framework Api调测记录总结
date: 2020-03-20 11:09
categories: ONAP
tags: 技术积累
description: 本文章主要接收ONAP整个构建观察中对应Policy组件的调测，保证各个组件正常，调用和查看各个Policy组件api，方便后续Policy调试部署。
---

# Policy Framework Api调测记录总结

## Policy Framework接口描述

在学习Policy过程中，一开始忽略了两个api文件：

PolicyAPI.postman_collection.json和PolicyAPI.postman_environment.json

重新学习api的这两个文件发现对于理解Policy Framework的整个服务有很多帮助。



比如里边包含了Policy Framework整体环境的健康检查api，可以由此判定PF框架所需的服务，并且检测是否正常。



以下服务为api接口：

```text
		{
			"key": "POLICY-API-URL",
			"value": "https://IP:PORT",
			"enabled": true
		},
		{
			"key": "POLICY-PAP-URL",
			"value": "https://IP:PORT",
			"enabled": true
		},
		{
			"key": "POLICY-APEX-URL",
			"value": "https://IP:PORT",
			"enabled": true
		},
		{
			"key": "POLICY-DROOLS-URL",
			"value": "https://IP:PORT",
			"enabled": true
		},
		{
			"key": "POLICY-XACML-URL",
			"value": "https://IP:PORT",
			"enabled": true
		},
		{
			"key": "DMAAP-URL",
			"value": "https://IP:PORT",
			"enabled": true
		}
```

目前调测不通的有Policy-Api、Policy-PAP、Policy-Apex、Policy-XACML、DMAAP



已有的wiki资料：《Apex 安装构建教程》 https://onap.readthedocs.io/en/latest/submodules/policy/parent.git/docs/apex/APEX-Install-Guide.html 

## 手动测试调用

Policy-PAP健康检查：

curl -k -v https://10.107.57.2:6969/policy/pap/v1/healthcheck -H 'Authorization: Basic aGVhbHRoY2hlY2s6emIhWHp0RzM0' -H 'X-ONAP-RequestID: b079fb6b-210c-4bf0-a992-f692c525220e'

 

Policy-Api接口：

curl -k -v https://10.96.142.54:6969/policy/api/v1/healthcheck -H 'Authorization: Basic aGVhbHRoY2hlY2s6emIhWHp0RzM0' -H 'X-ONAP-RequestID: b079fb6b-210c-4bf0-a992-f692c525220e'



Policy-Apex接口：

curl -k -v https://10.97.84.62:6969/policy/apex-pdp/v1/healthcheck -H 'Authorization: Basic aGVhbHRoY2hlY2s6emIhWHp0RzM0' -H 'X-ONAP-RequestID: b079fb6b-210c-4bf0-a992-f692c525220e'

**Drools引擎接口：(调试不通，授权不通过，错误信息：401 Unauthorized)**

curl -k -v https://10.107.43.80:6969/policy/pdpd/v1/healthcheck -H 'Authorization: Basic aGVhbHRoY2hlY2s6emIhWHp0RzM0' -H 'X-ONAP-RequestID: b079fb6b-210c-4bf0-a992-f692c525220e'

Polciy-XACML接口：

curl -k -v https://10.108.170.144:6969/policy/pdpx/v1/healthcheck -H 'Authorization: Basic aGVhbHRoY2hlY2s6emIhWHp0RzM0' -H 'X-ONAP-RequestID: b079fb6b-210c-4bf0-a992-f692c525220e'

PAP Statics接口：

curl -k -v https://10.107.57.2:6969/policy/pap/v1/statistics -H 'Authorization: Basic aGVhbHRoY2hlY2s6emIhWHp0RzM0' -H 'X-ONAP-RequestID: b079fb6b-210c-4bf0-a992-f692c525220e'

## 自动化测试

在linux下进行自动化测需要使用到newman工具，可以对ONAP社区提供的API进行自动化批量测试，比较方便，不用一个个测试，极大提高测试验证效率。

Policy Api的两个文件：

> PolicyAPI.postman_collection.json和PolicyAPI.postman_environment.json
>
> 分别为测试接口，和Policy各组件地址环境变量接口。



### 依赖安装

版本安装：nodejs（v12.16.1） 和npm（6.14.2）

首先newman依赖node和npm，安装npm

> apt install nodejs
>
> apt install npm

然后执行newman安装

> npm install -g newman

### 自动化测试Policy

首先修改XXXXpostman_environment.json文件

执行命令：

> newman run PolicyAPI.postman_collection.json -k -e PolicyAPI.postman_environment.json>>result.txt

### 报错汇总及解决

执行命令安装newman时，提示以下错误

~~~err
npm ERR! Linux 4.4.0-143-generic
npm ERR! argv "/usr/bin/nodejs" "/usr/bin/npm" "install" "-g" "newman"
npm ERR! node v4.2.6
npm ERR! npm  v3.5.2
npm ERR! code EAI_AGAIN
npm ERR! errno EAI_AGAIN
npm ERR! syscall getaddrinfo

npm ERR! getaddrinfo EAI_AGAIN registry.npmjs.org:443
npm ERR! 
npm ERR! If you need help, you may report this error at:
npm ERR!     <https://github.com/npm/npm/issues>

npm ERR! Please include the following file with any support request:
npm ERR!     /root/npm-debug.log

~~~

当时被gfw屏蔽了，需要配置国内的npm源，执行以下命令：

> npm config set registry http://10.12.26.4:8081/repository/npm-public/

配置之后出错，执行newman报错：

~~~err
npm ERR! Linux 4.4.0-143-generic
npm ERR! argv "/usr/bin/nodejs" "/usr/bin/npm" "install" "-g" "newman"
npm ERR! node v4.2.6
npm ERR! npm  v3.5.2
npm ERR! code E404

npm ERR! 404 Not found : repository
npm ERR! 404 
npm ERR! 404  'repository' is not in the npm registry.
npm ERR! 404 You should bug the author to publish it (or use the name yourself!)
npm ERR! 404 
npm ERR! 404 Note that you can also install from a
npm ERR! 404 tarball, folder, http url, or git url.

npm ERR! Please include the following file with any support request:
npm ERR!     /root/npm-debug.log

~~~

可能是由于npm源仓库不稳定，缺少东西，于是更换了cnpm

执行命令,重置官方源：

~~~cmd
npm config set registry https://registry.npmjs.org/
~~~

淘宝源：

~~~cmd
npm config set registry https://registry.npm.taobao.org/
~~~



发现不好使，59虚机上不了网络，安装失败，猝。

我们虚机需要配置dns就可以访问：

~~~cmd
 /etc/resolv.conf 文件配置
 
nameserver 211.136.17.107
nameserver 180.76.76.76
~~~



安装成功后，运行newman错误:

~~~cmd
module.js:328
    throw err;
    ^

Error: Cannot find module '/root/policy_api/node'
    at Function.Module._resolveFilename (module.js:326:15)
    at Function.Module._load (module.js:277:25)
    at Function.Module.runMain (module.js:442:10)
    at startup (node.js:136:18)
    at node.js:966:3

~~~

后续查到文章，nodejs需要>=v6,从官网下载node进行解压.

~~~cmd
npm -g install npm
~~~

 **查看node、npm版本** 

~~~cmd
node -v
v12.16.1
npm -v
6.14.2
~~~

再次执行newman安装，升级了下newman，问题得到解决。

**https访问证书问题**

~~~cmd
unable to get local issuer certificate
~~~

查询资料，配置ssl为忽略也不好使。

> 以下四种方法都无效：
>
> * npm config set strict-ssl false 
>
> * npm config set registry http://registry.npm.taobao.org/
>
> * npm install npm -g --ca=null 
>
> * npm config set ca="" 

最后在社区询问找到两种办法，一种是配置AAI的证书，一种是使用类似curl -k用法，忽略ssl错误即可。

最终找到newman的的用法，完整命令如下：

> newman run PolicyAPI.postman_collection.json -k -e PolicyAPI.postman_environment.json

# 参考文档

 https://blog.csdn.net/DwZ735660836/article/details/85682677  

 https://www.cnblogs.com/thelastman/p/9007568.html 

 https://blog.csdn.net/m0_37618247/article/details/83182795 

 https://www.cnblogs.com/kibana/p/10260026.html 