## OpenVPN官方文档



### 10.5.3. OpenVPN官方文档

上面章节大致介绍了通常的安装、配置方法，这里再介绍一下可能有用的（关于OpenVPN，不和CentOS相关），参考自[软件指南针](http://www.softown.cn/summary/openvpn-howto)翻译的[官方文档](https://openvpn.net/index.php/open-source/documentation/howto.html)。也可查看[OpenVPN相关文章](https://openvpn.net/index.php/open-source/articles.html)或[OpenVPN wiki](https://community.openvpn.net/openvpn)。

#### 10.5.3.1. 安装OpenVPN

[英文原文](http://openvpn.net/index.php/open-source/documentation/howto.html#install)

可以[在这里](https://openvpn.net/index.php/open-source/downloads.html)下载OpenVPN源代码和Windows安装程序。最近的版本(2.2及以后版本)也发布了Debian和RPM包(.deb和.rpm)。详情查看[OpenVPN wiki](https://community.openvpn.net/openvpn)。出于安全考虑，建议下载完毕后检查一下文件的[签名信息](https://openvpn.net/index.php/open-source/documentation/sig.html)。OpenVPN可执行文件提供了服务器和客户端的所有功能，因此服务器和客户端都需要安装OpenVPN的可执行文件。

**Linux版安装事项（使用RPM包管理工具）**

```
rpmbuild -tb openvpn-[version].tar.gz
```














































#### 10.5.3.1 服务器与客户端子网的互相访问

假设服务器网段为`192.168.0.0/24`，服务器IP地址为`192.168.0.5`，客户端网段为`192.168.111.0/24`，客户端IP地址为`192.168.111.10`（不能和服务器网段相同，如果有多个客户端，每个客户端网段必须是不同用户名和不同的网段）。

在服务器和客户端都开启IP和TUN/TAP转发（按照上一节安装好服务器好后已经开启，不过还需要设置防火墙）：

各操作系统[开启IP和TUN/TAP转发的设置](https://community.openvpn.net/openvpn/wiki/265-how-do-i-enable-ip-forwarding)：

**Windows**（[官网IPEnableRouter相关文章`https://technet.microsoft.com/en-us/library/cc962461.aspx`）](https://technet.microsoft.com/en-us/library/cc962461.aspx)）：

注册表编辑器`HKLM\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters`将`IPEnableRouter`的值改为`1`；运行`services.msc`将“Routing and Remote Access”设置为自动并启动（此步不知是否必须）；

检查：运行`ipconfig -all`，查看“Windows IP 配置”中显示`IP 路由已启用: 是`

防火墙无特别设置，可以先关闭防火墙，用ping测试是否连通会方便些，测试完成后再打开防火墙

**Linux**：

`echo 1 > /proc/sys/net/ipv4/ip_forward`

防火墙设置：iptables添加`-A FORWARD -d 192.168.111.0/24 -j ACCEPT`；firewalld运行`firewall-cmd --permanent --direct --passthrough ipv4 -t nat -I POSTROUTING -o eth0 -j MASQUERADE -s 192.168.111.0/24`

**OS X**：

`sudo sysctl -w net.inet.ip.forwarding=1`

未测试是否需要配置防火墙

开启转发后对服务器进行配置：

```
# 下面的注释只介绍作用
push "route 192.168.0.0 255.255.255.0"  # 客户端可访问服务器所在的192.168.0.0/24网段
client-config-dir ccd  # 可以对每个客户端进行单独的配置
```

在`ccd`目录下，以客户端用户名为名称创建文件，写入以下内容：

```
iroute 192.168.111.0 255.255.255.0
```

子网网段`192.168.111.0/24`路由到该用户客户端。

然后在**服务器配置文件**加入一行：

```
push "route 192.168.0.0 255.255.255.0"
client-config-dir ccd
route 192.168.111.0 255.255.255.0  # 服务端网段可访问客户端网段
```

`route`语句控制从系统内核到OpenVPN服务器的路由，`iroute`控制从OpenVPN服务器到远程客户端的路由（不是太懂，照做就可以了）。

**客户端网段之间的互相访问**

如果允许其他客户端访问此客户端的`192.168.111.0/24`网段，在服务器配置文件中加入：

```
client-to-client
push "route 192.168.111.0 255.255.255.0"
```

在其他终端**添加路由**，几种方式：

一、在网关添加路由。如在`192.168.0.0/24`网关处添加一条访问`192.168.111.0/24`时以`192.168.0.5`为网关，在`192.168.111.0/24`网关处添加一条访问`192.168.0.0/24`时以`192.168.111.10`为网关。

二、在其他终端上添加路由表。（服务器所在网段的）Windows如下添加：`route add 192.168.111.0 mask 255.255.255.0 192.168.0.5`，（客户端所在网段的）Windows如下添加：`route add 192.168.5.0 mask 255.255.255.0 192.168.111.10`，如果添加永久路由，使用`-P`参数。其它系统的自行网上找资料。

三、可以综合参考以上两种方式，来控制加入此互访网络的终端。
