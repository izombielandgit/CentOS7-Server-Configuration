







**个人建议的配置**：

`mkdir /var/www/tmp` # 创建一个临时目录，下面不放任何内容

编辑`/etc/httpd/conf/httpd.conf`文件：

```
# 在最后一行“IncludeOptional conf.d/*.conf”上面添加以下内容
<VirtualHost *:80>
   DocumentRoot /var/www/tmp
   ServerName www.srv.world
</VirtualHost>
```

`systemctl restart httpd`

如果未设置其他虚拟主机，通过IP或域名访问Web服务器是访问到默认的不存在的网页，然后再根据自己的需要设置虚拟主机


