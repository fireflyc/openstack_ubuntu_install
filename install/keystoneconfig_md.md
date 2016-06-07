# 配置
### 配置数据库
```
#新建数据库
CREATE DATABASE keystone;
#授权
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' \
  IDENTIFIED BY 'keystone123';
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' \
  IDENTIFIED BY 'keystone123';
```
### 配置keystone
禁用keystone自动启动，因为M版本的openstack采用apache作为WSGI Server不再使用自带的WSGI Server。(其实……更慢了，openstack不愿意背锅了，把球踢给Apache把问题归结为：用户自己“不会部署”)
```
echo manual | sudo tee /etc/init/keystone.override
```
生成随机数key用于配置keystone.conf中的admin_token
```
openssl rand -hex 10
```
admin_token就是通过上面指令生成的随机数（你乐意的话可以用任何字符串）
```
#编辑/etc/keystone/keystone.conf
[DEFAULT]
admin_token = a2a289f7cb74dfa88498
log_dir = /var/log/keystone #定义keystone的日志文件
[database]
connection = mysql+pymysql://keystone:keystone123@10.0.8.50/keystone #配置keystone数据库访问信息
[token]
provider = fernet #token的提供方式，fernet是一种对称加密方法基于AES-CBC Heroku项目御用，还有其他的比如UUID之类的。。
```
执行完上述配置之后，执行数据库同步就会自动在数据库中生成表结构，插入初始数据。
```
#同步数据库
su -s /bin/sh -c "keystone-manage db_sync" keystone
```
由于我们采用Fernet作为Token的提供方式，所以需要我们初始化Fernet的密钥。**这一步不能省略，否则后面访问Keystone会“授权失败”**
```
#初始化Fernet key
keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
```

### 配置Apache
背锅侠Apache作为一个老牌Web服务器，它的配置非常简单，我们配置apache开放5000和35357端口，然后配置连个HOST与之对应。他们对应的就是keystone的api。5000端口是Admin用户访问时的端口，35357端口是member用户访问的端口。**v3版本的新特性，v2版本只有5000端口。另外这个规定不是强制性的，admin用户也可以访问35357。只是部署上“建议”**
```
#编辑/etc/apache2/sites-available/wsgi-keystone.conf

Listen 5000
Listen 35357

<VirtualHost *:5000>
    WSGIDaemonProcess keystone-public processes=5 threads=1 user=keystone group=keystone display-name=%{GROUP}
    WSGIProcessGroup keystone-public
    WSGIScriptAlias / /usr/bin/keystone-wsgi-public
    WSGIApplicationGroup %{GLOBAL}
    WSGIPassAuthorization On
    ErrorLogFormat "%{cu}t %M"
    ErrorLog /var/log/apache2/keystone.log
    CustomLog /var/log/apache2/keystone_access.log combined

    <Directory /usr/bin>
        Require all granted
    </Directory>
</VirtualHost>

<VirtualHost *:35357>
    WSGIDaemonProcess keystone-admin processes=5 threads=1 user=keystone group=keystone display-name=%{GROUP}
    WSGIProcessGroup keystone-admin
    WSGIScriptAlias / /usr/bin/keystone-wsgi-admin
    WSGIApplicationGroup %{GLOBAL}
    WSGIPassAuthorization On
    ErrorLogFormat "%{cu}t %M"
    ErrorLog /var/log/apache2/keystone.log
    CustomLog /var/log/apache2/keystone_access.log combined

    <Directory /usr/bin>
        Require all granted
    </Directory>
</VirtualHost>
```
重启服务
```
#加载站点
ln -s /etc/apache2/sites-available/wsgi-keystone.conf /etc/apache2/sites-enabled
#重启apache
service apache2 restart
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

**后续所有针对openstack命令的操作都需要先执行 source adminrc**

