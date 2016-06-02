# 配置
## Controller节点
```
#便捷/etc/cinder/cinder.conf

```
### 验证
```
source adminrc
neutron ext-list
```

## Storage节点
### 安装软件包
```
apt-get install cinder-api cinder-scheduler python-cinderclient
```

