# 安装
## Controller节点
### 安装软件包
```
apt-get install cinder-api cinder-scheduler python-cinderclient
```

## Storage节点
### 安装软件包
```
apt-get install lvm2 cinder-volume python-mysqldb
```

```
pvcreate /dev/sdb
vgcreate cinder-volumes /dev/sdb
```
