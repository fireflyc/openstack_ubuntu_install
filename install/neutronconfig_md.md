# 配置

##Controller节点
```
编辑 /etc/neutron/neutron.conf

[DEFAULT]
#开启L2
core_plugin = ml2
#允许开启的L3-L7的服务，比如虚拟路由器，虚拟负载均衡，虚拟防火墙
service_plugins = router
#是否允许重叠子网
allow_overlapping_ips = True
rpc_backend = rabbit
auth_strategy = keystone
#看名字就知道意思了。。。
notify_nova_on_port_status_changes = True
notify_nova_on_port_data_changes = True

#下面是必须的，neutron调用OVS的时候是通过命令行完成的，它会执行sudo尝试切换到root用户执行，如果没有下面的配置你会看到ovs-agent满屏幕的错误信息
[agent]
root_helper = sudo /usr/bin/neutron-rootwrap /etc/neutron/rootwrap.conf

[database]
connection = mysql+pymysql://neutron:neutron123@10.0.8.50/neutron

[oslo_messaging_rabbit]
rabbit_host = controller.openstack
rabbit_userid = neutron_controller
rabbit_password = openstack123

[keystone_authtoken]
auth_uri = http://controller.openstack:5000
auth_url = http://controller.openstack:35357
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = neutron
password = neutron123

[nova]
auth_url = http://controller.openstack:35357
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = nova
password = nova123

```
Controller不需要开启OVS Agent，仅仅作为neutron-server，所以它的配置和网络节点很像（网络节点配置了外部网络）
```
#编辑/etc/neutron/plugins/ml2/ml2_conf.ini
[ml2]
type_drivers = flat,vlan,gre,vxlan
tenant_network_types = vxlan
mechanism_drivers = openvswitch,l2population
extension_drivers = port_security

[ml2_type_vxlan]
vni_ranges = 1:1000

[securitygroup]
enable_ipset = True
```
Controller节点的Nova组件（nova-api进程所在的节点），其中service_metadata_proxy会开启meta data代理，metadata_proxy_shared_secret需要和网络节点的meta data server配置一样，这里都设置为mate_data123
```
编辑/etc/nova/nova.conf 增加下面几行

[neutron]
url = http://controller.openstack:9696
auth_url = http://controller.openstack:35357
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = neutron
password = neutron123
service_metadata_proxy = True
metadata_proxy_shared_secret = mate_data123
```
同步数据库，重启服务
```
su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf \
  --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron

service nova-api restart
service neutron-server restart
```
### 验证
```
source adminrc
neutron ext-list
```

##Compute节点
##Network节点