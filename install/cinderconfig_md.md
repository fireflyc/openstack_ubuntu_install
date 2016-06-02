# 配置
## Controller节点
```
#便捷/etc/cinder/cinder.conf
[DEFAULT]
rpc_backend = rabbit
auth_strategy = keystone
my_ip = 10.0.8.52

[database]
connection = mysql+pymysql://cinder:cinder123@10.0.8.52/cinder

[oslo_messaging_rabbit]
rabbit_host = 10.0.5.50
rabbit_userid = openstack
rabbit_password = openstack123

[keystone_authtoken]
auth_uri = http://10.0.5.50:5000
auth_url = http://10.0.5.50:35357
memcached_servers = 10.0.5.50:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = cinder
password = cinder123

[oslo_concurrency]
lock_path = /var/lib/cinder/tmp
```
同步数据库，重启服务
```
cinder-manage db sync
```
配置Nova，修改控制节点的nova.conf
```
#编辑/etc/nova/nova.conf

[cinder]
os_region_name = RegionOne

```
重启nova服务，重启cinder服务
```
service nova-api restart
service cinder-scheduler restart
service cinder-api restart
```

## Storage节点
### 安装软件包
```
apt-get install cinder-api cinder-scheduler python-cinderclient
```

