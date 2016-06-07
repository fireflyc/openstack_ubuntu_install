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
rabbit_userid = openstack
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
首先修改系统参数，允许ip forward
```
#编辑/etc/sysctl.conf
net.ipv4.conf.all.rp_filter=0
net.ipv4.conf.default.rp_filter=0
```
执行
```
#刷新系统参数
sysctl -p 
```
```
#编辑/etc/neutron/neutron.conf
[DEFAULT]
verbose = True
rpc_backend = rabbit
auth_strategy = keystone
core_plugin = ml2
service_plugins = router
allow_overlapping_ips = True

[database]
connection = mysql+pymysql://neutron:neutron123@10.0.8.50/neutron

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
username = neutron
password = neutron123
```
```
#编辑/etc/neutron/plugins/ml2/openvswitch_agent.ini
[ovs]
local_ip = 10.10.10.1

[agent]
tunnel_types = vxlan
l2_population = True
prevent_arp_spoofing = True

[securitygroup]
enable_security_group = True
firewall_driver = neutron.agent.linux.iptables_firewall.OVSHybridIptablesFirewallDriver

```
允许nova使用neutron服务（在计算节点操作）
```
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
```
重启服务
```
service nova-compute restart
service openvswitch-switch restart
service neutron-openvswitch-agent restart
```



##Network节点
首先修改系统参数，允许ip forward
```
#编辑/etc/sysctl.conf
net.ipv4.ip_forward=1
net.ipv4.conf.all.rp_filter=0
net.ipv4.conf.default.rp_filter=0
```
执行
```
#刷新系统参数
sysctl -p 
```
```
#编辑/etc/neutron/neutron.conf
[DEFAULT]
rpc_backend = rabbit
auth_strategy = keystone
core_plugin = ml2
service_plugins = router
allow_overlapping_ips = True

[database]
connection = mysql+pymysql://neutron:neutron123@10.0.8.50/neutron

[oslo_messaging_rabbit]
rabbit_host = controller.openstack
rabbit_userid = openstack
rabbit_password = openstack123

[agent]
root_helper = sudo /usr/bin/neutron-rootwrap /etc/neutron/rootwrap.conf

[keystone_authtoken]
auth_uri = http://controller.openstack:5000
auth_url = http://controller.openstack:35357
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = neutron
password = neutron123
```
配置ml2
```
#编辑/etc/neutron/plugins/ml2/ml2_conf.ini
[ml2]
type_drivers = flat,vlan,gre,vxlan
tenant_network_types = vxlan
mechanism_drivers = openvswitch,l2population
extension_drivers = port_security

[ml2_type_vxlan]
#vxlan的范围
vni_ranges = 1:1000

[ml2_type_flat]
#外部网络是flat的类型
flat_networks = external

[securitygroup]
enable_ipset = True
```
开启OVS，其中local_ip是neutron用于overlay的网络IP
```
#编辑/etc/neutron/plugins/ml2/openvswitch_agent.ini
[ovs]
local_ip = 10.10.10.254
bridge_mappings = external:br-ex
[agent]
tunnel_types = vxlan
l2_population = True
prevent_arp_spoofing = True
[securitygroup]
enable_security_group = True
firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
```
开启L3 proxy external_network_bridge留空让所有网络使用一个agent。
**千万要记住，有两个参数external_network_bridge、gateway_external_network_id 都不要设置。不要听信文档上的瞎说，如果你设置了会发现“外部端口”不是连接到br-ex，而是连接到br-int，所以虚拟机没有办法访问外网。不设置的作用相当于external_network_bridgebr-ex，如果为空的作用相当于external_network_bridge=br-int。gateway_external_network_id用于多租户访问外网的时候的分流，具体介绍参考文档(虽然不靠谱)**
```
# 编辑/etc/neutron/l3_agent.ini
[DEFAULT]
interface_driver = neutron.agent.linux.interface.OVSInterfaceDriver
```
开启DHCP,dnsmasq_config_file选项指向dnsmasq-neutron.conf文件，该文件里面包含了强制设置DHCP Client使用Jumbo frames（加大MTU大小）
```
echo 'dhcp-option-force=26,1450' | sudo tee /etc/neutron/dnsmasq-neutron.conf
```

```
# 编辑/etc/neutron/dhcp_agent.ini
[DEFAULT]
interface_driver = neutron.agent.linux.interface.OVSInterfaceDriver
dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
enable_isolated_metadata = True
dnsmasq_config_file = /etc/neutron/dnsmasq-neutron.conf
```
metadata_proxy_shared_secret的值要和nova的一样
```
#编辑/etc/neutron/metadata_agent.ini
[DEFAULT]
nova_metadata_ip = 10.0.8.50
metadata_proxy_shared_secret = mate_data123
```

配置OVS桥接，其中br-ex要和配置文件里面的名字保持一致，p2p2是外部网络的网卡
```
ovs-vsctl add-br br-ex
ovs-vsctl add-port br-ex p2p2

#禁用Generic Receive Offload
ethtool -K p2p2 gro off
```
重启服务
```
service openvswitch-switch restart
service neutron-openvswitch-agent restart
service neutron-dhcp-agent restart
service neutron-metadata-agent restart
service neutron-l3-agent restart
```
### 验证
在Controller节点
```
neutron agent-list
```

