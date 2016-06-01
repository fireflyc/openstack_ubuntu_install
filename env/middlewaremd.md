# 安装中间件

mysql和rabbitmq是所有组件都需要的“中间件”。我们安装在Controller节点上。

## 安装需要的软件包
```
#安装mysql和python的mysqlsdk
apt-get install mariadb-server python-pymysql

#rabbitmq
apt-get install rabbitmq-server
```

## 配置
编辑Mysql配置文件
```
#编辑/etc/mysql/my.cnf
bind-address            = 0.0.0.0 #允许远程访问
default-storage-engine = innodb #默认引擎为innodb
innodb_file_per_table #innodb每个表一个数据文件（良好的习惯）
#设置数据库编码为utf-8
collation-server = utf8_general_ci 
character-set-server = utf8
max_connections = 4096
```
配置RabbitMQ，添加RabbitMQ，设置该用户访问所有vhost、exchange
```
#添加rabbitmq用户
rabbitmqctl add_user openstack openstack123
rabbitmqctl set_permissions openstack ".*" ".*" ".*"
```
如果要开启rabbitmq的web访问界面需要更改rabbitmq的配置
```
#编辑 /etc/rabbitmq/enabled_plugins (如果没有则新建)，下面配置结束的时候是一个“.”

[rabbitmq_management].
```
为openstack用户赋administrator，拥有web登录的权限
```
rabbitmqctl set_user_tags openstack administrator
```
然后就可以通过访问http的15672端口访问rabbitmq的web界面了。