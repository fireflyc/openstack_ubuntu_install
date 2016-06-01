# 初始化配置
Keystone有以下几个概念

>- User：顾名思义就是使用服务的用户，可以是人、服务或者是系统，只要是使用了 OpenStack 服务的对象都可以称为用户。
>- Tenant：租户，可以理解为一个人、项目或者组织拥有的资源的合集。在一个租户中可以拥有很多个用户，这些用户可以根据权限的划分使用租户中的资源。
>- Role：角色，用于分配操作的权限。角色可以被指定给用户，使得该用户获得角色对应的操作权限。
>- Token：指的是一串比特值或者字符串，用来作为访问资源的记号。Token 中含有可访问资源的范围和有效时间。
>- Service：服务，提供一组功能的组件。如Nova、Glance、Swift。
>- EndPoint: 服务暴露的接口，一般都是一个HTTP URL。可以包含“变量”（详见Nova的配置）

扯完概念之后继续看我们的安装，Keystone安装成功之后需要提供一个Service的租户用于管理OpenStack所有的Service，一个Admin租户还有一个可选的demo租户(建议配置)。

```
#keystone本身也是openstack的service，所以先把自己放进去。开通service器有两步：

#新建一个指定名字的service，
openstack service create \
  --name keystone --description "OpenStack Identity" identity
  
#新建三个不同用途的endpoint。
openstack endpoint create --region RegionOne \
  identity public http://controller.openstack:5000/v3
 openstack endpoint create --region RegionOne \
  identity internal http://controller.openstack:5000/v3
 openstack endpoint create --region RegionOne \
  identity admin http://controller.openstack:35357/v3

#新建admin租户、service租户、demo租户
openstack domain create --description "Default Domain" default
openstack project create --domain default \
  --description "Admin Project" admin
openstack user create --domain default \
  --password-prompt admin
openstack role create admin
openstack role add --project admin --user admin admin

#service
openstack project create --domain default \
  --description "Service Project" service

#demo
openstack project create --domain default \
  --description "Demo Project" demo
  
openstack user create --domain default \
  --password-prompt demo
  
openstack role create user

openstack role add --project demo --user demo user
```

## 验证
```
unset OS_TOKEN OS_URL
#测试admin
 openstack --os-auth-url http://controller:35357/v3 \
  --os-project-domain-name default --os-user-domain-name default \
  --os-project-name admin --os-username admin token issue
#测试demo
openstack --os-auth-url http://controller:5000/v3 \
  --os-project-domain-name default --os-user-domain-name default \
  --os-project-name demo --os-username demo token issue
```
## 设定脚本
脚本的用户是省去每次输入openstack命令时候的用户认证，新增一一个adminrc文件(位置随便放，我一般放在home目录)。脚本中有修改操作系统提示符，加载脚本之后会自动添加(os-admin)这样的提示
```
#adminrc
export OS_PROJECT_DOMAIN_NAME=default
export OS_USER_DOMAIN_NAME=default
export OS_PROJECT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=admin123
export OS_AUTH_URL=http://controller.openstack:35357/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
export PS1="\[\e]0;\u@\h: \w\a\]${debian_chroot:+($debian_chroot)}\u@\h(os-admin):\w\$"
```
使用脚本的时候只需要输入 source adminrc
