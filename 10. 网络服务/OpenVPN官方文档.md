# OpenVPN官方文档

上面章节大致介绍了通常的安装、配置方法，这里再介绍一下可能有用的（关于OpenVPN，不和CentOS相关），参考自[软件指南针](http://www.softown.cn/summary/openvpn-howto)翻译的[官方文档](https://openvpn.net/index.php/open-source/documentation/howto.html)。也可查看[OpenVPN相关文章](https://openvpn.net/index.php/open-source/articles.html)或[OpenVPN wiki](https://community.openvpn.net/openvpn)。

## 1. 安装OpenVPN

[英文原文](https://openvpn.net/index.php/open-source/documentation/howto.html#install)

可以[在这里](https://openvpn.net/index.php/open-source/downloads.html)下载OpenVPN源代码和Windows安装程序。最近的版本(2.2及以后版本)也发布了Debian和RPM包(.deb和.rpm)。详情查看[OpenVPN wiki](https://community.openvpn.net/openvpn)。出于安全考虑，建议下载完毕后检查一下文件的[签名信息](https://openvpn.net/index.php/open-source/documentation/sig.html)。OpenVPN可执行文件提供了服务器和客户端的所有功能，因此服务器和客户端都需要安装OpenVPN的可执行文件。

**Linux安装事项（使用RPM包管理工具）**

如果你使用的Linux发行版支持RPM包管理工具，例如：RedHat、CentOS、Fedora、SUSE等。最好使用这种方法来安装。最简单的方法就是找到一个可以在当前Linux发行版上使用的二进制RPM文件。你也可以使用如下命令创建(build)你自己的二进制RPM文件：

```
rpmbuild -tb openvpn-[version].tar.gz
```

有了`.rpm`格式的文件，就可以使用如下常规命令来安装它。

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

官方版的OpenVPN Windows安装程序自带[OpenVPN GUI](https://community.openvpn.net/openvpn/wiki/OpenVPN-GUI)，OpenVPN GUI允许用户通过一个托盘程序来管理OpenVPN连接。也可以使用其他的[GUI程序](https://community.openvpn.net/openvpn/wiki/OpenVPN-GUI)。

安装OpenVPN之后，OpenVPN将会与扩展名为`.ovpn`的文件进行关联。想要运行OpenVPN，可以：

* 右键点击OpenVPN配置文件`.ovpn`，在弹出的关联菜单中选择【Start OpenVPN on this configuration file】即可使用该配置文件启动OpenVPN。运行OpenVPN后，可以使用快捷键`F4`来退出程序。
* 以命令提示符的方式运行OpenVPN，例如：`openvpn myconfig.ovpn`。运行之后，同样可以使用快捷键`F4`来退出VPN。
* 在OpenVPN安装路径`/config`目录（一般默认为`C:\Program Files\OpenVPN\config`）中放置一个或多个`.ovpn`格式的配置文件，然后启动名为OpenVPN Service的Windows服务。你可以点击【开始】->【控制面板】->【管理工具】->【服务】，从而进入Windows服务管理界面。

**Mac OS X安装事项**

Angelo Laub和Dirk Theisen已经开发出了[OpenVPN GUI for OS X](https://tunnelblick.net/)。

**其他操作系统**

通常情况下，其他操作系统也可以使用`./configure`方法来安装OpenVPN，或者也可以自行查找一个适用于你的操作系统/发行版的OpenVPN接口或安装包。

```
./configure
make
make install
```

更多安装说明参阅[这里](https://openvpn.net/index.php/open-source/documentation/install.html?start=1)。

## 2. 选择使用路由还是桥接

[英文原文](https://openvpn.net/index.php/open-source/documentation/howto.html#vpntype)

查看“路由 VS. 以太网桥接”的[FAQ](https://community.openvpn.net/openvpn/wiki/FAQ#bridge1)。也可以查看OpenVPN[以太网桥接](https://openvpn.net/index.php/open-source/documentation/miscellaneous/76-ethernet-bridging.html)页面以了解关于桥接的更多事项和细节。

总的来说，对于大多数人而言，“路由模式”可能是更好的选择，因为它可以比“桥接模式”更高效更简单地搭建一个VPN（基于OpenVPN自带的配置）。路由模式还提供了更强大的对指定客户端的访问权限进行控制的能力。

推荐使用“路由模式”，除非你需要用到只有“桥接模式”才具备的某些功能特性，例如：

* VPN需要处理非IP协议，例如IPX。
* 在VPN网络中运行的应用程序需要用到网络广播(network broadcasts)，例如：局域网游戏。
* 你想要在没有Samba或WINS服务器的情况下，能够通过VPN浏览Windows文件共享。

## 3. 设置私有子网

[英文原文](https://openvpn.net/index.php/open-source/documentation/howto.html#numbering)

创建一个VPN需要借助私有子网将不同地方的成员连接在一起。

互联网号码分配机构(IANA)专为私有网络保留了以下三块IP地址空间(制定于RFC 1918规范中)：

* 10.0.0.0-10.255.255.255(10/8 prefix)
* 172.16.0.0-172.31.255.255(172.16/12 prefix)
* 192.168.0.0-192.168.255.255(192.168/16 prefix)

在VPN配置中使用这些地址。对选择IP地址并将IP地址冲突或子网冲突的发生概率降到最低而言，这一点非常重要。以下是需要避免的冲突类型：

* VPN中不同的网络场所使用相同的局域网子网编号所产生的冲突。
* 远程访问连接自身使用的私有子网与VPN的私有子网发生冲突。

简而言之，处于不同局域网的客户端和客户端之间，它们自身所在的局域网IP段不要发生冲突；客户端自身所在的局域网IP段也不要和VPN使用的局域网IP段发生冲突。

举个例子，假设你使用了流行的`192.168.0.0/24`作为你的VPN子网。现在，你尝试在一个网吧内连接VPN，该网吧的WIFI局域网使用了相同的子网。这将产生一个路由冲突，因为你的计算机不知道`192.168.0.1`是指本地WIFI的网关，还是指VPN上的相同地址。

再举个例子，假设你想通过VPN将多个网络场所连接在一起，但是每个网络场所都使用了`192.168.0.0/24`作为自己的局域网子网。如果你不添加一个复杂的NAT翻译层，它们将无法工作。因为这些网络场所没有唯一的子网来标识它们自己，VPN不知道如何在多个网络场所之间路由数据包。

最佳的解决方案是避免使用`10.0.0.0/24`或者`192.168.0.0/24`作为VPN的私有子网。相反，你应该使用一些在你可能连接VPN的场所（例如咖啡厅、酒店、机场）不太可能使用的私有子网。最佳的候选者应该是在浩瀚的`10.0.0.0/8`网络块中间选择一个子网（例如：`10.66.77.0/24`）。

总的来说，为了避免跨多个网络场所的IP编号冲突，请始终使用唯一的局域网子网编号。

笔者注：在某些情况下使用相同的IP网段也是可以使用的，不过最好是尽可能避免冲突。

## 4. 创建证书

[英文原文](https://openvpn.net/index.php/open-source/documentation/howto.html#pki)

创建过程略。

证书相关文件

|文件名|谁需要|作用|是否需保密|
|-|-|-|-|
|ca.crt|服务器 + 所有客户端|根CA证书|NO|
|ca.key|密钥签名机|根CA密钥|YES|
|dh{n}.pem|服务器|迪菲·赫尔曼参数|NO|
|server.crt|服务器|服务器证书|NO|
|server.key|服务器|服务器密钥|YES|
|client1.crt|client1|Client1证书|NO|
|client1.key|client1|Client1密钥|YES|
|client2.crt|client2|Client2证书|NO|
|client2.key|client2|Client2密钥|YES|































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
