# 教你如何在centos7配置ipv6地址

#### 本文教你如何在[centos7](https://so.csdn.net/so/search?q=centos7&spm=1001.2101.3001.7020)系统上面配置ipv6地址

#### 第一步，修改 /etc/modprobe.d/disable_ipv6.conf

```
cp /etc/modprobe.d/disable_ipv6.conf /etc/modprobe.d/disable_ipv6.conf_backup ##先备份原始配置
vi /etc/modprobe.d/disable_ipv6.conf
```

- 将options ipv6 disable等于1变为0

```
options ipv6 disable=0
```

#### 第二步，修改 /etc/sysconfig/network

```
cp /etc/sysconfig/network /etc/sysconfig/network_backup #备份
vi /etc/sysconfig/network
```

- 将NETWORKING_IPV6=no变为yes

```
PEERNTP=no
NETWORKING_IPV6=yes
GATEWAY=139.255.255.0
```

#### 第三步，修改 /etc/sysctl.conf

```
cp /etc/sysctl.conf /etc/sysctl.conf_backup #备份
vi /etc/sysctl.conf
```

- 添加部分内容，就是把disable的选项都等于0

```
net.ipv6.conf.all.disable_ipv6 = 0
net.ipv6.conf.default.disable_ipv6 = 0
net.ipv6.conf.lo.disable_ipv6 = 0
```

#### 第四步，修改 /etc/sysconfig/network-scripts/ifcfg-eth0

```
cp /etc/sysconfig/network-scripts/ifcfg-eth0 /etc/sysconfig/network-scripts/ifcfg-eth0_backup #备份
vi /etc/sysconfig/network-scripts/ifcfg-eth0
```

- 主要是新增IPV6ADDR和IPV6_DEFAULTGW两部分，网址是我自己copy别人的

```
IPV6INIT="yes"
IPV6_AUTOCONF="yes"
IPV6_DEFROUTE="yes"
IPV6_FAILURE_FATAL="no"
IPV6_ADDR_GEN_MODE="stable-privacy"
IPV6ADDR=2001:250:4000:2000::53 #重要
IPV6_DEFAULTGW=2001:250:4000:2000::1#重要
```

#### 最后一步，创建系统在启动时自动加载 IPv6 模块的脚本

```
vi /etc/sysconfig/modules/ipv6.modules
```

- 脚本内容

```
!/bin/sh
	if [ ! -c /proc/net/if_inet6 ] ; then
	exec /sbin/insmod /lib/modules/uname -r/kernel/net/ipv6/ipv6.ko
	fi
```

#### 重启系统，加载 IPv6 模块

- 重启系统

```
reboot
```

- 查看ipv6地址的输出

```
ifconfig |grep -i inet6
```

- 输出**inet6 2001:250:4000:2000::53 prefixlen 64 scopeid 0x0/ global**表示ipv地址添加成功！

```
inet6 2001:250:4000:2000::53  prefixlen 64  scopeid 0x0<global>
inet6 fe80::afb4:6574:86eb:880  prefixlen 64  scopeid 0x20<link>
inet6 ::1  prefixlen 128  scopeid 0x10<host>
```



[原文](https://blog.csdn.net/Jason160918/article/details/100121138)