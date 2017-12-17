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

关于证书、密钥安全性的问题可查看原文。

## 4. 创建配置文件

[英文原文](https://openvpn.net/index.php/open-source/documentation/howto.html#config)

**取得示例配置文件**

最好使用[OpenVPN示例配置文件](https://openvpn.net/index.php/open-source/documentation/howto.html#examples)作为配置起点。这些文件可以在下列地方找到：

* OpenVPN源代码版的`sample-config-files`目录。
* 如果通过RPM或DEB来安装OpenVPN，则为`/usr/share/doc/packages/openvpn`或`/usr/share/doc/openvpn`中的`sample-config-files`目录。
* Windows系统中的【开始】->【所有程序】->【OpenVPN】->【Shortcuts】->【OpenVPN Sample Configuration Files】，也就是OpenVPN安装路径`/sample-config`目录。

注意：在Linux、BSD或类Unix系统中，示例配置文件叫做`server.conf`和`client.conf`。在Windows中叫做`server.ovpn`和`client.ovpn`。

**编辑服务器端配置文件**

服务器端配置文件示例是OpenVPN服务器端配置的最佳起始点。

该示例配置将使用一个虚拟的TUN网络接口（路由模式），在UDP端口1194（OpenVPN的官方端口号）监听远程连接，并从子网`10.8.0.0/24`中为连接的客户端分配虚拟地址。

在使用示例配置文件之前，先编辑`ca`、`cert`、`key`、`dh`参数，将它们分别指向对应文件。

此时，服务器端配置文件是可用的，但可以进一步自定义该文件：

* 如果你使用的是[以太网桥接](https://openvpn.net/index.php/open-source/documentation/miscellaneous/76-ethernet-bridging.html)模式，必须使用`server-bridge`和`dev tap`指令来替代`server`和`dev tun`指令。
* 如果想让你的OpenVPN服务器监听一个TCP端口，而不是UDP端口，使用`proto tcp`替代`proto udp`（如果想同时监听UDP和TCP端口，必须启动两个单独的OpenVPN实例）。
* 如果你想使用`10.8.0.0/24`范围之外的虚拟IP地址，修改`server`指令。记住，虚拟IP地址范围必须是一个你当前网络未使用的私有范围。
* 如果你想让通过VPN连接的客户端和客户端之间能够互访，启用`client-to-client`指令（去掉注释）。默认情况下，客户端只能够访问服务器。
* 如果你正在使用Linux、BSD或类Unix操作系统，你可以使启用`user nobody`和`group nobody`指令来提高安全性。

如果想要在同一计算机上运行多个OpenVPN实例，每一个示例都应该使用不同的配置文件。可能存在下列情形：

* 每个示例使用不同的端口号（UDP和TCP协议使用不同的端口空间，因此你可以运行一个后台进程并同时监听UDP-1194和TCP-1194）。
* 如果你使用的是Windows系统，每个OpenVPN配置都需要有自己的TAP-Windows适配器。你可以通过【开始】->【所有程序】->【TAP-Windows】->【Add a new TAP-Windows virtual ethernet adapter】来添加一个额外的适配器。
* 如果你运行多个相同目录的OpenVPN实例，请确保编辑那些会创建输出文件的指令，以便于多个实例不会覆盖掉对方的输出文件。这些指令包括`log`、`log-append`、`status`和`ifconfig-pool-persist`。

**编辑客户端配置文件**

客户端配置示例文件（在Linux/BSD/Unix中为`client.conf`，在Windows中为`client.ovpn`）参照了服务器配置示例文件的默认指令设置。

* 与服务器配置文件类似，首先编辑`ca`、`cert`和`key`参数，使它们指向对应文件。注意，每个客户端都应该有自己的证书/密钥对。只有`ca`文件是OpenVPN和所有客户端通用的。
* 下一步，编辑`remote`指令，将其指向OpenVPN服务器的主机名/IP地址和端口号（如果OpenVPN服务器在防火墙或NAT网关之后的单网卡机器上运行，请使用网关的公网IP地址，和你在网关中配置的转发到OpenVPN服务器的端口号）。
* 最后，确保客户端配置文件和用于服务器配置的指令保持一致。主要检查dev（dev或tap）和proto（udp或tcp）指令是否一致。此外，如果服务器和客户端配置文件都使用了`comp-lzo`和`fragment`指令，也需要保持一致。

## 5. 启动VPN并测试

**启动服务器**

[英文原文](https://openvpn.net/index.php/open-source/documentation/howto.html#start)

首先，确保OpenVPN服务器能够正常连接网络。这意味着：

* 开启防火墙上的UDP-1194端口（或者你配置的其他端口）。
* 或者，创建一个端口转发规则，将防火墙/网关的UDP-1194端口转发到运行OpenVPN服务器的计算机上。

下一步，[确保TUN/TAP接口没有被防火墙屏蔽](https://community.openvpn.net/openvpn/wiki/FAQ#firewall)。

为了简化故障排除，最好使用命令行来初始化启动OpenVPN服务器（或者在Windows上右击.ovpn文件），而不是以后台进程或服务的方式启动：

```
openvpn [服务器配置文件]
```

正常的服务器启动应该像这样：

```
Sun Feb  6 20:46:38 2005 OpenVPN 2.0_rc12 i686-suse-linux [SSL] [LZO] [EPOLL] built on Feb  5 2005
Sun Feb  6 20:46:38 2005 Diffie-Hellman initialized with 1024 bit key
Sun Feb  6 20:46:38 2005 TLS-Auth MTU parms [ L:1542 D:138 EF:38 EB:0 ET:0 EL:0 ]
Sun Feb  6 20:46:38 2005 TUN/TAP device tun1 opened
Sun Feb  6 20:46:38 2005 /sbin/ifconfig tun1 10.8.0.1 pointopoint 10.8.0.2 mtu 1500
Sun Feb  6 20:46:38 2005 /sbin/route add -net 10.8.0.0 netmask 255.255.255.0 gw 10.8.0.2
Sun Feb  6 20:46:38 2005 Data Channel MTU parms [ L:1542 D:1450 EF:42 EB:23 ET:0 EL:0 AF:3/1 ]
Sun Feb  6 20:46:38 2005 UDPv4 link local (bound): [undef]:1194
Sun Feb  6 20:46:38 2005 UDPv4 link remote: [undef]
Sun Feb  6 20:46:38 2005 MULTI: multi_init called, r=256 v=256
Sun Feb  6 20:46:38 2005 IFCONFIG POOL: base=10.8.0.4 size=62
Sun Feb  6 20:46:38 2005 IFCONFIG POOL LIST
Sun Feb  6 20:46:38 2005 Initialization Sequence Completed
```

**启动客户端**

和服务器端配置一样，最好使用命令行来初始化启动OpenVPN客户端（在Windows上，也可以直接右击client.ovpn文件），而不是以后台进程或服务的方式来启动。

```
openvpn [客户端配置文件]
```

Windows上正常的客户端启动看起来与前面服务器端的输出非常相似，并且应该以`Initialization Sequence Completed`信息作为结尾。

尝试从客户端通过VPN发送`ping`命令。

如果你使用的是路由模式（例如：在服务器配置文件中设置`dev tun`），运行：

```
ping 10.8.0.1
```

如果使用桥接模式（例如：在服务器配置文件中设置`dev tap`），尝试ping一个服务器端的以太网子网中的IP地址。

如果能够`ping`成功，那么就连接成功了。

**故障排除**

如果`ping`失败或者无法完成OpenVPN客户端的初始化，这里列出了一个常见问题以及对应解决方案的清单：

1、得到错误信息：`TLS Error: TLS key negotiation failed to occur within 60 seconds (check your network connectivity)`。该错误表明客户端无法与服务器建立一个网络连接。

解决方案：

* 确保客户端配置中使用的服务器主机名/IP地址和端口号是正确的。
* 如果OpenVPN服务器所在计算机只有单个网卡，并处于受保护的局域网内，请确保服务器端的网关防火墙使用了正确的端口转发规则。举个例子，假设你的OpenVPN服务器在某个局域网内，IP为`192.168.4.4`，并在UDP端口1194上监听客户端连接。服务于子网`192.168.4.x`的NAT网关应该有一个端口转发规则，该规则将公网IP地址的UDP端口1194转发到`192.168.4.4`。
* 打开服务器防火墙，允许外部连接通过UDP-1194端口（或者在服务器配置文件中设置的其他端口）。

2、得到错误信息：`Initialization Sequence Completed with errors`。该错误发生在：(a)你的Windows系统没有一个正在运行的DHCP客户端服务，(b)或者你在XP SP2上使用了某些第三方的个人防火墙。

解决方案：

* 启动DHCP客户端服务器，并确保在XP SP2系统中使用的是一个工作正常的个人防火墙。

3、得到信息`Initialization Sequence Completed`但是`ping`测试失败。这通常意味着服务器或客户端的防火墙屏蔽了TUN/TAP接口，从而阻塞了VPN网络。

解决方案：

* 禁用客户端TUN/TAP接口上的防火墙（如果存在的话）。以Windows XP SP2为例，你可以进入【Windows安全中心】->【Windows 防火墙】->【高级】，并取消选中TAP-Windows适配器前面的复选框（从安全角度来说，禁止客户端防火墙屏蔽TUN/TAP接口通常是合理的，这是在告诉防火墙不要阻止可信的VPN流量）。此外，也需要确保服务器的TUN/TAP接口没有被防火墙屏蔽（不得不说的是，选择性地设置服务器端TUN/TAP接口的防火墙有助于提高一定的安全性，请参考访问策略部分）。
* 笔者注：也有可能本身已连通但是`ping`的主机防火墙阻止了ICMP，可以进行相关设置后再尝试

4、当采用UDP协议的配置启动时，出现连接中断，并且服务器日志文件显示下行：

```
TLS: Initial packet from x.x.x.x:x, sid=xxxxxxxx xxxxxxxx
```

但客户端的日志文件不会显示相同的信息。

解决方案：

* 你只有一个从客户端到服务器的单向连接。而从服务器到客户端的方向则被（通常是客户端这边的）防火墙阻挡了。该防火墙可能是运行于客户端的个人软件防火墙，或是客户端的NAT路由网关。请修改防火墙以允许从服务器到客户端的UDP数据包返回。

想了解更多额外的故障排除信息，请查看[FAQ](https://community.openvpn.net/openvpn/wiki/FAQ)。






















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
