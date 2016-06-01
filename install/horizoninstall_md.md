# Horizon安装与配置

Horizon就是OpenStack提供的操作UI（丑哭了，难用哭了）。它的安装很简单，基本上就是安装一下软件，配置一下apache，重启apache完事。

本章介绍在Controller节点上安装Horizon

##安装软件
```
apt-get install openstack-dashboard
```
##配置
```
#修改/etc/openstack-dashboard/local_settings.py
OPENSTACK_HOST='10.0.8.50'
```
##重启服务
```
service apache2 reload
service apache2 restart
```
访问地址 http://10.0.8.50/horizon/

