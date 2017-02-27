







**个人建议的配置**：

`mkdir /var/www/tmp` # 创建一个临时目录，下面不放任何内容

编辑`/etc/httpd/conf/httpd.conf`文件：

```
# 在最后一行“IncludeOptional conf.d/*.conf”上面添加以下内容
<VirtualHost *:80>
   DocumentRoot /var/www/tmp
   ServerName www.srv.world
<Directory /var/www/tmp>
   AllowOverride None
   Options FollowSymLinks
   Require all denied
</Directory>
</VirtualHost>
```

`systemctl restart httpd`

如果未设置其他虚拟主机，通过IP或域名访问Web服务器默认访问不到网页，然后再根据自己的需要设置虚拟主机。

编辑`/etc/httpd/conf.d/vhost.conf`文件：

```
<VirtualHost *:80>
   DocumentRoot /home/cent/public_html
   ServerName www.virtual.host
   ServerAdmin webmaster@virtual.host
   ErrorLog logs/virtual.host-error_log
   CustomLog logs/virtual.host-access_log combined
<Directory /home/cent/public_html>
   AllowOverride None
   Options FollowSymLinks
   Require all denied
</Directory>
</VirtualHost>
```




















