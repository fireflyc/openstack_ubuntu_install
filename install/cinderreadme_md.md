# Cinder

Cinder是OpenStack复杂块存储（Block Storage）的模块，它主要为Nova提供

## 涉及到的服务器
角色 |配置项目
---|---
控制节点|安装准备(配置数据库和keystone)、安装配置neutron-server
网络节点|安装配置neutron-ovs-agent l3-agent(路由器) dhcp-agent(DHCP) metadata-agent(元数据，实现自动配置)
计算节点|安装配置ovs-agent

## 配置数据库
```
CREATE DATABASE neutron;
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost'  IDENTIFIED BY 'neutron123';
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%'  IDENTIFIED BY 'neutron123';
```

## 配置Keystone账号
```
#新建用户
openstack user create --domain default --password-prompt neutron

openstack role add --project service --user neutron admin

openstack service create --name neutron \
  --description "OpenStack Networking" network

openstack endpoint create --region RegionOne \
  network public http://controller.openstack:9696
  
openstack endpoint create --region RegionOne \
  network internal http://controller.openstack:9696

openstack endpoint create --region RegionOne \
  network admin http://controller.openstack:9696 