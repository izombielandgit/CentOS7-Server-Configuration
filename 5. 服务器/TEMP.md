### 5.1.15 PHP + PHP-FPM

[先安装PHP](#513-使用php脚本)

`yum -y install php-fpm` # 安装PHP-FPM

配置Apache httpd：

编辑`/etc/httpd/conf.d/php.conf`文件：

```
# 作如下更改
<FilesMatch \.php$>
#    SetHandler application/x-httpd-php
    SetHandler "proxy:fcgi://127.0.0.1:9000" 
</FilesMatch>
```

```
systemctl start php-fpm
systemctl enable php-fpm
systemctl restart httpd
```

`echo '<?php phpinfo(); ?>' > /var/www/html/info.php` # 创建phpinfo


访问它，如果显示“FPM/FastCGI”则表示安装成功：

![httpd-php-fpm-phpinfo](../Contents/httpd-php-fpm-phpinfo.png)































