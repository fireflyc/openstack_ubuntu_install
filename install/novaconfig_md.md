# 配置

## Controller节点

```
#编辑 /etc/nova/nova.conf
[DEFAULT]
#此处可为天坑，log_dir已经过期了，log-dir是建议的配置项。但是nova-compute根本不是识别log-dir，只认log_dir。你没有听过，是同样的openstack版本，同样都是nova组件。
log-dir=/var/log/nova
#服务启动之后的状态文件(用于kill服务)
state_path=/var/lib/nova
#防止启动多个实例
lock_path=/var/lock/nova
#采用RabbitMQ作为通讯中间件
rpc_backend = rabbit
#用户认证是keystone
auth_strategy = keystone
#开启neutron
use_neutron = True
#意思就是——我不要防火墙(我们不是是nova-compute，所有确实不需要)
firewall_driver = nova.virt.firewall.NoopFirewallDriver
#配置上controller的ip地址（运行nova-api进程的节点）
my_ip = 10.0.8.50

[database]
connection = mysql+pymysql://nova:nova123@controller.openstack/nova

[api_database]
#新版本的nova有两个数据库，一个叫nova，一个是新增的nova_api
connection = mysql+pymysql://nova:nova123@controller.openstack/nova_api

[vnc]
#设置为novnc进程所在的服务器地址，此处为controller
vncserver_listen = 10.0.8.50
vncserver_proxyclient_address = 10.0.8.50

#连接rabbitmq的配置信息
[oslo_messaging_rabbit]
rabbit_host = controller.openstack
rabbit_userid = openstack
rabbit_password = openstack123

[keystone_authtoken]
auth_uri = http://controller.openstack:5000
auth_url = http://controller.openstack:35357
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = nova
password = nova123

#防止启动多实例
[oslo_concurrency]
lock_path = /var/lib/nova/tmp
```
同步数据
```
#该语句没有输出，不报错就是对的。囧
su -s /bin/sh -c "nova-manage api_db sync" nova

su -s /bin/sh -c "nova-manage db sync" nova
```
重启服务
```
service nova-api restart
service nova-consoleauth restart
service nova-scheduler restart
service nova-conductor restart
service nova-novncproxy restart
```

## Compute节点
```
编辑 /etc/nova/nova.conf
[DEFAULT]
#注意此处是log_dir不是log-dir
log_dir=/var/log/nova
state_path=/var/lib/nova
lock_path=/var/lock/nova
rpc_backend = rabbit
auth_strategy = keystone
my_ip = 10.0.8.52
use_neutron = True
#由neutron提供了
firewall_driver = nova.virt.firewall.NoopFirewallDriver
resume_guests_state_on_host_boot = true

[oslo_messaging_rabbit]
rabbit_host = controller.openstack
rabbit_userid = openstack
rabbit_password = openstack123

[keystone_authtoken]
auth_uri = http://controller.openstack:5000
auth_url = http://controller.openstack:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = nova
password = nova123

[vnc]
enabled = True
#在计算节点启动vnc server，由控制节点的novnc进程来访问
vncserver_listen = 0.0.0.0
vncserver_proxyclient_address = 10.0.8.52
#控制节点的novnc
novncproxy_base_url = http://10.0.8.50:6080/vnc_auto.html

[glance]
api_servers = http://controller.openstack:9292

[oslo_concurrency]
lock_path = /var/lib/nova/tmp
```
重启服务
```
service nova-compute restart
```

## 验证
在Controller节点

```
source adminrc
nova service-list
```

