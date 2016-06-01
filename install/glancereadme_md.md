# Glance
Glance是OpenStack中负责镜像管理的功能组件，虚拟机新建完成之后Nova会通过Glance下载镜像然后把镜像被当做虚拟机的“硬盘”使用。

本章介绍如何在Controller上安装Glance。
## 涉及到的服务器
角色 |配置项目
---|---
控制节点|所有

## 配置数据库
```
CREATE DATABASE glance;
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' \
  IDENTIFIED BY 'glance123';
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' \
  IDENTIFIED BY 'glance123';
```
## 配置Keystone账号
```
openstack user create --domain default --password-prompt glance

openstack role add --project service --user glance admin

openstack service create --name glance \
  --description "OpenStack Image" image
  
openstack endpoint create --region RegionOne \
  image public http://controller.openstack:9292
  
openstack endpoint create --region RegionOne \
  image internal http://controller.openstack:9292
  
openstack endpoint create --region RegionOne \
  image admin http://controller.openstack:9292
```
以下所有内容都在Controller节点上完成