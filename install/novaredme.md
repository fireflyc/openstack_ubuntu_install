# Nova

Nov是OpenStack中的计算组件，它的底层是调用的KVM的API。你可以理解为它KVM的一个“包装”。

本章主要讲解如何在Compute节点上安装并配置Nova（注意上下文提示命令是在哪台服务器上进行的操作）

## 涉及到的服务器
角色 |配置项目
---|---
控制节点|安装准备(配置数据库和keystone)、安装配置nova-api nova-conductor nova-consoleauth nova-novncproxy nova-scheduler
计算节点|安装配置nova-compute

**下面的内容在Controller节点完成，主要用于初始化数据库和keystone的用户**

## 配置数据库
```
CREATE DATABASE nova_api;
CREATE DATABASE nova;
GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'localhost' \
  IDENTIFIED BY 'nova123';
GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'%' \
  IDENTIFIED BY 'nova123';
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' \
  IDENTIFIED BY 'nova123';
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' \
  IDENTIFIED BY 'nova123';
```
## 配置Keystone账号
```
 #配置租户
openstack user create --domain default \
  --password-prompt nova
#输入nova123

openstack role add --project service --user nova admin
 
 #配置service、endpoint
openstack service create --name nova \
  --description "OpenStack Compute" compute
 
openstack endpoint create --region RegionOne \
  compute public http://controller.openstack:8774/v2.1/%\(tenant_id\)s
  
openstack endpoint create --region RegionOne \
  compute internal http://controller.openstack:8774/v2.1/%\(tenant_id\)s

openstack endpoint create --region RegionOne \
  compute admin http://controller.openstack:8774/v2.1/%\(tenant_id\)s
```