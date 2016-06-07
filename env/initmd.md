# 初始化服务器
本章所有的内容配置和软件要求在所有节点上进行，主要用于解决“中国特色”问题。

## 修改ubuntu安装源(加速)
由于众所周知的原因，Ubuntu默认的源速度非常的慢，所以这里更换阿里云的源。（163的也可以，但是速度没有阿里云的快）
```
# 备份历史源
mv /etc/apt/sources.list /etc/apt/sources.list.bak
```
```
deb http://mirrors.aliyun.com/ubuntu/ trusty main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ trusty-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ trusty-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ trusty-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ trusty main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ trusty-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ trusty-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ trusty-backports main restricted universe multiverse
```
如果不执行这一步后面的添加仓库动作可能会失败。
```
apt-get update && apt-get dist-upgrade
```

## 添加openstack仓库
如果执行add-apt-repository或update失败则尝试删除/etc/apt/source.list.d/下面的mitaka.list文件之后重新添加。
```
add-apt-repository cloud-archive:mitaka
apt-get update && apt-get dist-upgrade
#不安装这个无法使用openstack这个命令
apt-get install python-openstackclient
```