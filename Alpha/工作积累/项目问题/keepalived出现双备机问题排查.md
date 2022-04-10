# keepalived启动出现一直是备机状态

## 问题现象

两个主机的keepalived设置匹配号为1后启动，keepalived状态都是running，但是浮动ip不可用  ，ping不通VRRP虚拟IP地址。

## 问题排查

```
Sep 10 09:36:36 localhost systemd: Started IPv4 firewall with iptables.
Sep 10 09:36:36 localhost Keepalived_vrrp[35249]: (vi): ip address associated with VRID 1 not present in MASTER advert : 172.16.98.233
Sep 10 09:36:36 localhost Keepalived_vrrp[35249]: bogus VRRP packet received on ens33 !!!
Sep 10 09:36:36 localhost Keepalived_vrrp[35249]: VRRP_Instance(vi) ignoring received advertisment...
Sep 10 09:36:39 localhost Keepalived_vrrp[35249]: (vi): ip address associated with VRID 1 not present in MASTER advert : 172.16.98.233
```

通过观察keepalived日志信息发现vrrpd实例进程报错，VRID冲突，keepalived使用VRRP协议，规定vrrpd实例唯一，即同一网段中virtual_router_id的值不能重复，否则会出错。



keepalive master的virtual_router_id和局域网内的其它的keepalive master的virtual_router_id有冲突，修改 /etc/keepalived/keepalived.conf 配置文件里的 virtual_router_id 1 为其它数值后，启动keepalive后日志不再报错，并且恢复正常状态。