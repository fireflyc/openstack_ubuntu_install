# Glance
Glance是OpenStack中负责镜像管理的功能组件，虚拟机新建完成之后Nova会通过Glance下载镜像然后把镜像被当做虚拟机的“硬盘”使用。

本章介绍如何在Controller上安装Glance。
### 涉及到的服务器
角色 |配置项目
---|---
控制节点|所有
