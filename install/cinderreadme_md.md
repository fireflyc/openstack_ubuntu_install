# Cinder

Cinder是OpenStack复杂块存储（Block Storage）的模块，它通过使用iSCSI, 光纤通道或者NFS协议，以及若干私有协议提供后端连接，展现给计算层（Nova）。

## 涉及到的服务器
角色 |配置项目
---|---
控制节点|安装准备(配置数据库和keystone)、安装配置cinder-api、cinder-scheduler 
存储节点|可以是单独的存储节点，在我们的实验环境中我们用网络节点作为我们的存储节点

**请在host文件中添加storage.openstack**

以下操作在controller节点完成

## 配置数据库
```
CREATE DATABASE cinder;
GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'localhost' IDENTIFIED BY 'cinder123';
GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'%'  IDENTIFIED BY 'cinder123';
```

## 配置Keystone账号
```
#其中storage.openstack是存储节点
openstack user create --domain default --password cinder123 --email cinder@example.com cinder

openstack role add --project service --user cinder admin

openstack service create --name cinder \
--description "OpenStack Block Storage" volume

openstack service create --name cinderv2 \
--description "OpenStack Block Storage" volumev2

openstack endpoint create --region RegionOne \
volume public http://storage.openstack:8776/v1/%\(tenant_id\)s

openstack endpoint create --region RegionOne \
volume internal http://storage.openstack:8776/v1/%\(tenant_id\)s

openstack endpoint create --region RegionOne \
volume admin http://storage.openstack:8776/v1/%\(tenant_id\)s

openstack endpoint create --region RegionOne \
volumev2 public http://storage.openstack:8776/v2/%\(tenant_id\)s

openstack endpoint create --region RegionOne \
volumev2 internal http://storage.openstack:8776/v2/%\(tenant_id\)s

openstack endpoint create --region RegionOne \
volumev2 admin http://storage.openstack:8776/v2/%\(tenant_id\)s
```
