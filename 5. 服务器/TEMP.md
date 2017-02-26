### 5.1.21. 配置mod_evasive

启用mod_evasive模块来防御DoS攻击等。

`yum --enablerepo=epel -y install mod_evasive` # 从EPEL安装

编辑`/etc/httpd/conf.d/mod_evasive.conf`文件：

```
# 每页间隔相同页面的请求数量的阈值
DOSPageCount   5

# 每个站点间隔相同侦听器上的同一客户端对任何对象的请求总数的阈值
DOSSiteCount   50

# 页计数阈值的时间间隔
DOSPageInterval   1

# 站点计数阈值的时间间隔
DOSSiteInterval   1

# 如果将客户端添加到阻止列表中，则客户端将被阻止的时间量（以秒为单位）
DOSBlockingPeriod   300

# 如果IP地址被列入黑名单，通知邮件地址
DOSEmailNotify   root@localhost

# 指定日志目录
DOSLogDir   "/var/log/mod_evasive"
```

```
mkdir /var/log/mod_evasive
chown apache. /var/log/mod_evasive
systemctl restart httpd
```

使用RPM软件包中包含的测试工具进行测试：

`perl /usr/share/doc/mod_evasive-*/test.pl`

```
HTTP/1.1 200 OK
HTTP/1.1 200 OK
HTTP/1.1 200 OK
HTTP/1.1 200 OK
HTTP/1.1 200 OK
.....
.....
HTTP/1.1 403 Forbidden  # 如果被阻止，转到“403 Forbidden”
HTTP/1.1 403 Forbidden
HTTP/1.1 403 Forbidden
.....
.....
HTTP/1.1 403 Forbidden
```

`ll /var/log/mod_evasive`

```
total 4
-rw-r--r-- 1 apache apache 5 Aug  5 15:42 dos-127.0.0.1
```

如果设置了通知，则发送如下：

`mail`

```
Heirloom Mail version 12.5 7/5/10.  Type ? for help.
"/var/spool/mail/root": 1 message 1 new
>N  1 Apache                Wed Aug  3 19:42  20/673
& 1
Message  1:
From apache@www.srv.world  Wed Aug  3 19:42:55 2015
Return-Path: <apache@www.srv.world>
X-Original-To: root@localhost
Delivered-To: root@localhost.srv.world
Date: Wed, 05 Aug 2015 15:42:54 +0900
To: root@localhost.srv.world
User-Agent: Heirloom mailx 12.5 7/5/10
Content-Type: text/plain; charset=us-ascii
From: apache@www.srv.world (Apache)
Status: R

To: root@localhost
Subject: HTTP BLACKLIST 127.0.0.1

mod_evasive HTTP Blacklisted 127.0.0.1
```