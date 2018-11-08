---
layout:     post
title:      授人以渔：Zabbix监控AC篇
subtitle:   以H3C AC为例
date:       2018-11-07
author:     MoKe
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - Zabbix
    - Items
    - Triggers
---

## 前言
通过Zabbix来监控AC的CPU、内存以及AP的CPU、内存与在线终端数。


## 正文
* 明确需要监控的设备类型，协议选择，以及需要取得的内容；
* 查找相关模板，网上有些制作好的模板，download下来删减即可使用；
* 如该模板没有你想要的监控项，可以查找oid自己来添加；
* 设置触发器，图表，聚合图形等。


Leader说：no date ，no bb

小声BB：若不知道自己想要什么，只会跌跌撞撞撞撞撞撞...



### 需求
内网有一套PRTG的监控系统，界面很清晰，但一直没有管理权，是代理商搭建的设备，最近替换新的AC，升级到V7版本，OID值都已经改变，监控处于不可用状态。

#### 确定以有的监控项
![](https://ws4.sinaimg.cn/large/006tNbRwly1fx0zcggjd0j31kw0kmq81.jpg)


#### 整理所需的监控项
|Device|监控项|OID|
|:-:|:-:|:-:|
|AC|CPU使用率|1.3.6.1.4.1.25506.2.6.1.1.1.1.6.97|
||内存使用率|1.3.6.1.4.1.25506.2.6.1.1.1.1.8.97|
||在线AP数量|1.3.6.1.4.1.25506.2.75.1.1.2.1.0|
||当前用户总数||
|AP|CPU实时利用率|1.3.6.1.4.1.25506.2.75.2.1.10.1.2.{#SNMPINDEX}|
||内存实时利用率|1.3.6.1.4.1.25506.2.75.2.1.10.1.4.{#SNMPINDEX}|
||实时终端数量|1.3.6.1.4.1.25506.2.75.2.1.2.1.7.{#SNMPINDEX}|
||Radio1资源利用率|1.3.6.1.4.1.25506.2.75.2.1.3.1.26.{#SNMPINDEX}.1|
||Radio2资源利用率|1.3.6.1.4.1.25506.2.75.2.1.3.1.26.{#SNMPINDEX}.2|
||Radio3资源利用率|1.3.6.1.4.1.25506.2.75.2.1.3.1.26.{#SNMPINDEX}.3|

#### 找轮子
通过Zabbix来监控现有的设备，而又没有相关经验，时间精力和智商都不允许我琢磨造轮子，所以善用搜索，用前人做好的轮子，修修补补再用三年。

[H3C wireless AC(chinese)](https://share.zabbix.com/network_devices/other/h3c-wireless-ac-chinese)



#### 修轮子
将文档下载下来，导入到系统，由于该模板创建时间为2016年9月，是H3C的V5版本，导致大部分的值不能被读出数据。

"No Such Object available on this agent at this OID"这个提示表明OID是不可用的。我们需要找到合适的OID填充进去。

![](https://ws3.sinaimg.cn/large/006tNbRwly1fx10d42hfxj31kw051gnf.jpg)

简单讲一下我查找OID的挫折历程：

1、	找H3C售后客服拿到OID值；

2、	新增OID值到Zabbix中，发现读不到数据；

3、	找售后客服，表示mib表没问题，是我的姿势不对，又丢给我一份mib库；

4、	通过Getif软件将mib库导入，通过SNMP连接到H3C的设备上；

5、	查找相应的跟，然后根据mib表来找到相关设备，确定OID值；

6、	找到相关值以后，添加到图表，然后执行，得出来的值和设备show出来的一致。

举个简单的例子，H3C提供的mib表中，CPU的oid值为

	1.3.6.1.4.1.25506.2.6.1.1.1.1.6

![](https://ws4.sinaimg.cn/large/006tNbRwly1fx10slh8mpj31kw0kle2a.jpg)

![](https://ws1.sinaimg.cn/large/006tNbRwly1fx10yxapohj319q0p2109.jpg)

![](https://ws2.sinaimg.cn/large/006tNbRwly1fx110keg5yj31kw0lz7c0.jpg)

值一致，所以.97的OID为cpu的正确的OID值。

	1.3.6.1.4.1.25506.2.6.1.1.1.1.6.97
	
将该OID值修改成.97，会发现已经可以读出数据了。

![](https://ws4.sinaimg.cn/large/006tNbRwly1fx1138a83bj31d40qin0o.jpg)

![](https://ws3.sinaimg.cn/large/006tNbRwly1fx115ow0cxj31kw0ojdnw.jpg)

OID值的问题，已经和H3C的工程师确认过，通过第三方的监控设备确实是需要加.97的。

这样，最起码证明，我们的思路没错。

有了OID值，就可以解决其余的问题了。

新的需求，可以按照根据模板来克隆修改。


### 最后
附上我的轮子
