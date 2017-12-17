# OpenVPN官方文档

上面章节大致介绍了通常的安装、配置方法，这里再介绍一下可能有用的（关于OpenVPN，不和CentOS相关），参考自[软件指南针](http://www.softown.cn/summary/openvpn-howto)翻译的[官方文档](https://openvpn.net/index.php/open-source/documentation/howto.html)。也可查看[OpenVPN相关文章](https://openvpn.net/index.php/open-source/articles.html)或[OpenVPN wiki](https://community.openvpn.net/openvpn)。

## 1. 安装OpenVPN

[英文原文](http://openvpn.net/index.php/open-source/documentation/howto.html#install)

可以[在这里](https://openvpn.net/index.php/open-source/downloads.html)下载OpenVPN源代码和Windows安装程序。最近的版本(2.2及以后版本)也发布了Debian和RPM包(.deb和.rpm)。详情查看[OpenVPN wiki](https://community.openvpn.net/openvpn)。出于安全考虑，建议下载完毕后检查一下文件的[签名信息](https://openvpn.net/index.php/open-source/documentation/sig.html)。OpenVPN可执行文件提供了服务器和客户端的所有功能，因此服务器和客户端都需要安装OpenVPN的可执行文件。

**Linux安装事项（使用RPM包管理工具）**

如果你使用的Linux发行版支持RPM包管理工具，例如：RedHat、CentOS、Fedora、SUSE等。最好使用这种方法来安装。最简单的方法就是找到一个可以在当前Linux发行版上使用的二进制RPM文件。你也可以使用如下命令创建(build)你自己的二进制RPM文件：

```
rpmbuild -tb openvpn-[version].tar.gz
```

有了`.rpm`格式的文件，你就可以使用如下常规命令来安装它。

```
rpm -ivh openvpn-[details].rpm
```

或者升级现有的OpenVPN版本：

```
rpm -Uvh openvpn-[details].rpm
```

安装OpenVPN的RPM包，需要如下这些依赖软件包：

* openssl （SSL协议及相关内容的开源实现）
* lzo （无损压缩算法）
* pam （身份验证模块）

此外，如果自己创建(build)二进制RPM包，需要如下几个依赖：

* openssl-devel
* lzo-devel
* pam-devel

可以查看[openvpn.spec](https://openvpn.net/index.php/open-source/documentation/install.html#rpm)文件，该文件包含了在RedHat Linux 9上创建RPM包，或者在减少依赖的情况下创建RPM包的更多信息。

**Linux安装事项（非RPM）**

如果你使用的系统是Debian、Gentoo或其他不基于RPM的Linux发行版，你可以使用当前发行版指定的软件包管理机制，例如Debian的apt-get或者Gentoo的emerge。

```
apt-get install openvpn  # 使用apt-get安装OpenVPN
emerge openvpn  # 使用emerge安装OpenVPN
```

也可以使用常规的`./configure`方法在安装Linux上安装OpenVPN。首先，解压`.tar.gz`文件：

```
tar xfz openvpn-[version].tar.gz
```

然后跳转到OpenVPN的顶级目录（`top-level directory`实际上就是OpenVPN解压后的目录），输入：

```
./configure
make
make install
```

通过`./configure`方式进行OpenVPN的编译安装之前，仍然需要先安装OpenVPN的依赖软件包openssl、lzo、pam。详细信息参考[Linux版OpenVPN安装、配置教程]()。

**Windows安装事项**

对Windows系统而言，可以直接在下载exe格式的可执行文件来安装OpenVPN。OpenVPN只能够在Windows XP及以上版本的系统中运行。还要注意，必须具备管理员权限才能够安装运行OpenVPN（该限制是Windows自身造成的，而不是OpenVPN）。安装OpenVPN之后，你可以用Windows后台服务的方式启动OpenVPN来绕过该限制；在这种情况下，非管理员用户也能够正常访问VPN。你可以点击查看[关于OpenVPN + Windows权限问题的更多讨论](http://openvpn.se/files/howto/openvpn-howto_run_openvpn_as_nonadmin.html)。





























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
