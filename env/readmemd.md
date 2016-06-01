# 准备环境

我们的环境一共有三个节点，控制节点（Controller）、网络节点（Netowkr）、计算节点（Compute）。OpenStack的网络采用Overlay网络（也有翻译为覆盖网络，不如不翻译），它支持两种模式——GRE和VxLAN，在我们的实验环境中选择用VxLAN。如果要修改为GRE模式其实配置基本上差不多，只需要修改一两个配置项就可以了。“附录: VxLAN介绍”

注意：
**所有的ip地址规划和主机名规划请自行修改，要求主机名和ip地址之间做hosts映射。（请保持所有服务器上host配置文件一致）**

**本文所有操作都是通过root进行的，你可以通过sudo passwd root来给root设置一个密码，这样就可以通过su切换到root了**
