---
title: HCSA
date: 2024-01-21 23:26:14
tags:
---

ACL: access comtrol list 访问控制列表是一种基于包过滤的访问控制技术，它可以根据设定的条件对接口上的数据包进行过滤，允许其通过或丢弃。
五元组: 源IP、源端口,目的IP、目的端口、协议号
DPI: 深度报文检测（Deep Packet Inspection）是一种网络流量分析技术，用于检测和过滤网络数据包中的内容。它可以对数据包的头部和有效载荷进行深入分析，以识别特定的应用程序、协议或行为模式。

二层 vswitch1
三层 trust-vr

StoneOS系统架构
● Zones
  ○ L2 Zone
  ○ L3 Zone
● Interfaces  绑定到安全域
  ○ 一个接口只能绑定到一个安全域
  ○ 一个安全域可以包含一个或多个接口
  ○ 绑定到三层安全域的接口才可以配置IP地址及管理服务
● Virtual Switch  转发二层数据
● Virtual Router  转发三层数据
  ○ 以上均可创建多个,相互独立,各自维护自己的MAC表、路由表
  ○ 设备默认关闭多VRouter功能,开启此功能需要设备重启生效
● Policy  
  ○ 经过FW的流量,都需要做安全策略的放行
    ■ 通过直连访问FW接口的不需要经过安全策略
    ■ 二层arp报文,虽然是经过FW的流量,不需要经过安全策略
  ○ 安全策略仅限于二层之间、三层之间,没有二三层之间的! 
  ○ FW自身产生的流量不需要进行安全策略的放行

FW部署:路由模式、透明模式、旁路模式、混合模式
CLI常用命令
show命令
show version
show interface
show ip route
show snat
show dnet
show policy
show configuration
save					//保存当前配置信息 webui是自动保存 重启设备配置仍存在
unset all             				//恢复出厂设置

接口配置
configure  								//进入全局配置视图
interface ethernet0/1     //进入0/1口
zone trust								//配置三层安全域
ip address 192.168.10.12	//配置ip

路由基本配置
configure
ip vrouter trust-vr				//进入policy配置视图
ip router 10.18.0.0/16 10.1.1.1
exit


配置安全策略
configure
policy-global				//进入ploicy配置shitu
rule from any to any service any permit


shutdown      // 关闭接口--》down
no shutdown   // 开启接口--》up

数据转发

为了达到上网访问Internet
● 接口
● 路由
● NAT
● 策略