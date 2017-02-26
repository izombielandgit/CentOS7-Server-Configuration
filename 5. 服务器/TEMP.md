### 5.1.17 配置mod_proxy

#### 5.1.17.1. 正向代理

启用mod_proxy模块以配置正向代理设置。

mod_proxy包含在httpd包中，默认启用，因此可以快速配置：

`grep "mod_proxy" /etc/httpd/conf.modules.d/00-proxy.conf` # 模块默认启用

```
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_ajp_module modules/mod_proxy_ajp.so
.....
LoadModule proxy_http_module modules/mod_proxy_http.so
LoadModule proxy_scgi_module modules/mod_proxy_scgi.so
```

编辑`/etc/httpd/conf.d/f_proxy.conf`文件：

```
<IfModule mod_proxy.c>
    # 正向代理功能On
    ProxyRequests On
    <Proxy *>
        # 访问权限
        Require ip 127.0.0.1 10.0.0.0/24
    </Proxy>
</IfModule>
```

编辑`/etc/httpd/conf/httpd.conf`文件：

```
# 更改监听端口
Listen 8080
```

`systemctl restart httpd`

firewalld防火墙设置，添加上面设置的端口（8080/TCP）：

```
firewall-cmd --add-port=8080/tcp --permanent
firewall-cmd --reload
```

如果启用了SELinux，更改布尔值：

`setsebool -P httpd_can_network_relay on`

[在客户端上配置代理客户端设置](https://www.server-world.info/en/note?os=CentOS_7&p=squid&f=2)，并确保可以正常访问任何网站。

#### 5.1.17.2. 反向代理

启用mod_proxy模块以配置反向代理设置。本例基于以下环境：

```
(1) www.srv.world       [10.0.0.31]    - Web Server#1
(2) node01.srv.world    [10.0.0.51]    - Web Server#2
```

本例配置将(1)Web服务器的请求转发到(2)Web服务器。

mod_proxy包含在httpd包中，默认启用，因此可以快速配置：

`grep "mod_proxy" /etc/httpd/conf.modules.d/00-proxy.conf` # 模块默认启用

```
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_ajp_module modules/mod_proxy_ajp.so
.....
LoadModule proxy_http_module modules/mod_proxy_http.so
LoadModule proxy_scgi_module modules/mod_proxy_scgi.so
```

编辑`/etc/httpd/conf.d/r_proxy.conf`文件：

```
<IfModule mod_proxy.c>
    ProxyRequests Off
    <Proxy *>
        Require all granted
    </Proxy>
    # 后端服务器和转发路径
    ProxyPass / http://node01.srv.world/
    ProxyPassReverse / http://node01.srv.world/
</IfModule>
```

`systemctl restart httpd`

访问前端服务器以确保后端服务器响应如下所示：

![httpd-mod-proxy-reverse1](../Contents/httpd-mod-proxy-reverse1.png)

可以配置负载均衡设置：

```
(1) www.srv.world       [10.0.0.31]    - Web Server#1
(2) node01.srv.world    [10.0.0.51]    - Web Server#2
(3) node02.srv.world    [10.0.0.52]    - Web Server#3
```

本例配置将(1)Web服务器的http请求转发到(2)Web服务器和(3)Web服务器：

编辑`/etc/httpd/conf.d/r_proxy.conf`文件：

```
<IfModule mod_proxy.c>
    ProxyRequests Off
    <Proxy *>
        Require all granted
    </Proxy>
    # 指定使用“lbmethod”进行负载均衡的方式。也可以设置“bytraffic”。
    ProxyPass / balancer://cluster lbmethod=byrequests
    <proxy balancer://cluster>
        BalancerMember http://node01.srv.world/ loadfactor=1
        BalancerMember http://node02.srv.world/ loadfactor=1
    </proxy>
</IfModule>
```

`systemctl restart httpd`

访问前端服务器以确保后端服务器响应如下所示：

![httpd-mod-proxy-reverse2](../Contents/httpd-mod-proxy-reverse2.png)