# Nova

Nov是OpenStack中的计算组件，它的底层是调用的KVM的API。你可以理解为它KVM的一个“包装”。

本章主要讲解如何在Compute节点上安装并配置Nova（注意上下文提示命令是在哪台服务器上进行的操作）

## 涉及到的服务器
角色 |配置项目
---|---
控制节点|安装准备(配置数据库和keystone)、安装配置nova-api nova-conductor nova-consoleauth nova-novncproxy nova-scheduler
计算节点|安装配置nova-compute