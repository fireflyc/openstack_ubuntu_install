# 配置

## 配置Glance
glance由两个进程组成

>- glance-api 一个WSGI服务，暴露API接口供外部调用。它不直接访问数据库响应API的时候它调用glance-registry读取数据库获取镜像列表。
>- glance-registry 向 glance-api 提供元数据数据库操作 REST API。

这种看似“鸡肋”的设计是有特殊原因的，它可以把两个进程分开部署，实现HA。

配置glance-api进程
```
#编辑配置文件
/etc/glance/glance-api.conf
[DEFAULT]

[database]
#一个Python的ORM框架
backend = sqlalchemy
connection = mysql+pymysql://glance:glance123@10.0.8.50/glance 

[glance_store]
#支持本地文件和http两种存储方式
stores = file,http
#默认是本地文件
default_store = file
#定义本地文件路径（镜像都放在这里）
filesystem_store_datadir = /var/lib/glance/images/

[image_format]
#支持的镜像格式，全部写上
disk_formats = ami,ari,aki,vhd,vmdk,raw,qcow2,vdi,iso,root-tar

#glance需要访问keystone，这里给出访问keystone api的时候的需要信息。
[keystone_authtoken]
auth_uri = http://controller.openstack:5000
auth_url = http://controller.openstack:35357
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = glance
password = glance123

[paste_deploy]
#告诉glance使用keystone作为验证工具（CLI的时候使用）
flavor = keystone
```
配置glance-registry进程(你没有看错，配置很像。少了glance_store和image_format)
```
#编辑/etc/glance/glance-registry.conf
[DEFAULT]

[database]
backend = sqlalchemy
connection = mysql+pymysql://glance:glance123@10.0.8.50/glance

[keystone_authtoken]
auth_uri = http://controller.openstack:5000
auth_url = http://controller.openstack:35357
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = openstack
password = openstack123

[paste_deploy]
flavor = keystone
```
同步数据库、重启服务
```
#同步数据库
su -s /bin/sh -c "glance-manage db_sync" glance
#重启服务
service glance-registry restart
service glance-api restart
```

## 验证
此处OpenStack官方文档上是错的，openstack命令是不能用到，通过debug我发现它访问了一个不知道什么鬼的路径。具体原因懒得分析了，直接使用glance命令。
```
wget http://download.cirros-cloud.net/0.3.4/cirros-0.3.4-x86_64-disk.img

#上传镜像
glance image-create --name "cirros-0.3.4-x86_64" \
--file cirros-0.3.4-x86_64-disk.img \
--disk-format qcow2 --container-format bare \
--visibility public --progress

#查看
glance image-list
```