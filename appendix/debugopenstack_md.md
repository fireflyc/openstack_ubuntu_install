# OpenStack生存指南

在安装和使用OpenStack的过程中我们总是会碰到各种各样的问题，如果想解决这些问题我们首先必须搞清楚OpenStack各个组件的关系。这里我画了一幅图，用来表达OpenStack中各个进程和所在的节点（部署的情况不一样所以你的环境下可能和我的图不太一样，注意区分一下）
![OpenStack进程图](../_image/openstack_process.png)
核心组件包括keystone，nova，neutron，glance，cinder也勉强算核心（没有cinder可以正常使用，只是云硬盘的功能缺失）。

Controller节点上的进程主要负责各个模块的API、管理、调度
* keystone是OpenStack中唯一一个没有做进程切分的。所以它只有一个进程——keystoneall
* cinder-api是Cinder的API，Controller是作为API的汇聚节点
* neutron-server是负责neutron的API，转换成内部的调用通过MQ下发给ovs-agent、L3、DHCP、matedate。


glance的进程被拆分为负责API和存储镜像、负责数据库的
* glance-api负责API和存储镜像，它收到用户请求之后转发给registry完成数据库操作。（通过HTTP接口而不是MQ接口，OpenStack所有的内部通讯都是MQ，这个是特例。我也不知道为什么。。。）如果有上传或者下载Image，该进程会负责写入和读取；
* glance-registry只是负责读写数据库，完成元数据操作(记录镜像的名字、上传时间等数据)；


Compute节点则只保留ovs-agent、nova-compute、cinder-volumne这三种执行实际工作的进程
Network节点则是ovs-agent和L3、DHCP
