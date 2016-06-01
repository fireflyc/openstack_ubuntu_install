OpenStack部署手册——mitaka(Ubuntu)
============
[OpenStack](https://www.openstack.org) OpenStack是一个美国国家航空航天局和Rackspace贡献给开源社区的云计算软件，它的授权许可是Apache。原始模块中包含“计算”、“存储”、“网络”三个模块，主要为企业提供私有云解决方案。现在的模块则更加丰富，而且应用范围也不局限于私有云。

可以把OpenStack理解为一个“粘合剂”，它把OpenVSwtich、Linux、LVM（Linux卷管理）、KVM（Linux内核虚拟化）等云计算相关的工具“粘合”在一起，提供了一套完整的，可以和VMWare Esxi抗衡的“云计算套件”。有意思的是，OpenStack本身也是用了著名了“胶水语言”——Python写的。

OpenStack的开源模式充满商业的味道，它不像Linux、Mysql那样是由一群英雄主导团结整个社区，沿着“英雄们”的方向前进。它更像是一个充斥着商业、混子、英雄的混合世界，它的规模也是空前的，开发人员的数量之大、水平差异，工具之复杂，分工之细，管理之规范已经远远超过任何一个开源软件。（没错，包括Kernel）

正是由于这种组成“超大规模开源协作开发”造成了OpenStack生命力无限（有很多商业公司支持，非常强大的生态圈），也造成了“OpenStack之殇”——非常复杂。这也就是写这个手册的目的，一般很少有哪个软件的部署会复杂到用一篇文章甚至一本书来阐述。

最新版本在线阅读：[GitBook](https://www.gitbook.com/book/fireflyc/openstack-ubuntu)。

本书源码在 Github 上维护，欢迎参与： [https://github.com/fireflyc/openstack_ubuntu_install](https://github.com/fireflyc/openstack_ubuntu_install)。



## 更新历史:
* V0.1: 2016-06-1
	*第一次提交。


## 参加步骤
* 在 GitHub 上 `fork` 到自己的仓库，如 `user/openstack_ubuntu_install `，然后 `clone` 到本地，并设置用户信息。

```
$ git clone git@github.com:user/openstack_ubuntu_install.git
$ cd openstack_ubuntu_install
$ git config user.name "User"
$ git config user.email user@email.com
```

* 修改代码后提交，并推送到自己的仓库。

```
$ #do some change on the content
$ git commit -am "Fix issue #1: change helo to hello"
$ git push
```

* 在 GitHub 网站上提交 pull request。
* 定期使用项目仓库内容更新自己仓库内容。

```
$ git remote add upstream https://github.com/fireflyc/openstack_ubuntu_install
$ git fetch upstream
$ git checkout master
$ git rebase upstream/master
$ git push -f origin master
```

**觉得我写的不错，请我喝咖啡**

![支付宝](http://o83khlyy2.bkt.clouddn.com/2.png)


**欢迎关注公众账号了解更多信息**

![公众红账号](http://o83khlyy2.bkt.clouddn.com/qrcode_for_gh_f3674ea54fee_258%20%281%29.jpg)
