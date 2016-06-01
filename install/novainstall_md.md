# 安装


## Controller节点
Controller上面有nova的几组管理和辅助进程，我们尽量保持计算节点的“精简”
>- nova-api提供API服务
>- nova-conductor nova-compute不直接访问数据库而是通过MQ把请求转给nova-conductor，由它来访问数据库
>- nova-scheduler 调度器，用户开通虚拟机的时候会自动选择一个“合适的”计算节点
>- nova-consoleauth 控制台认证
>- nova-novncproxy 控制台VNC（支持HTML5）


### 安装软件包
```
apt-get install nova-api nova-conductor nova-consoleauth \
  nova-novncproxy nova-scheduler
```

## Compute节点
计算节点只保留nova-compute进程
### 安装软件包
```
apt-get install nova-compute sysfsutils
```