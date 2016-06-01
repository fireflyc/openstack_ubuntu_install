# 准备环境

我们的环境一共有三个节点，控制节点（Controller）、网络节点（Netowkr）、计算节点（Compute）。OpenStack的网络采用Overlay网络（也有翻译为覆盖网络，不如不翻译），它支持两种模式——GRE和VxLAN，在我们的实验环境中选择用VxLAN。如果要修改为GRE模式其实配置基本上差不多，只需要修改一两个配置项就可以了。“附录: VxLAN介绍”

