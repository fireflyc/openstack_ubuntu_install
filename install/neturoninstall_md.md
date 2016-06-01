# 安装

## Controller节点
### 安装软件包
```
apt-get install neutron-server neutron-plugin-ml2 python-neutronclient
```
##Compute节点
### 安装软件包
```
apt-get install neutron-openvswitch-agent
```

##Network节点
### 安装软件包
网络节点承担dhcp、metadata、L3外部互联的功能
```
apt-get install -y neutron-plugin-ml2 neutron-openvswitch-agent \
neutron-l3-agent neutron-dhcp-agent neutron-metadata-agent
```