# Neutorn

一篇文章很难说得清楚SDN是什么，解决什么问题的。况且如果没有任何网络知识也无法理解SDN。所以这个问题够一两本书的分量了，大家自行学习。（推荐“土专家”的《SDN原理解析 转控分离的SDN架构》）

**对Neutron不熟悉请参考 附录: Neutron精要**

Neutron采用VxLAN作为Overlay网络，底层网络需要开启“Jumbo frames”支持9000bytes（vxlan传递的是1450byte的frame）的frame **一般都已经开启**

## 涉及到的服务器
角色 |配置项目
---|---
控制节点|安装准备(配置数据库和keystone)、安装配置neutron-server
网络节点|安装配置neutron-ovs-agent l3-agent(路由器) dhcp-agent(DHCP) metadata-agent(元数据，实现自动配置)
计算节点|安装配置ovs-agent

**下面的内容在Controller节点完成，主要用于初始化数据库和keystone的用户**

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
```
