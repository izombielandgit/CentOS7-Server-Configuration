### 5.1.19. 配置mod_ratelimit

启用mod_ratelimit模块以限制客户端的带宽。

mod_ratelimit包含在httpd包中，因此可以快速配置。

编辑`/etc/httpd/conf.modules.d/00-base.conf`文件：

```
# 取消注释
LoadModule ratelimit_module modules/mod_ratelimit.so
```

编辑`/etc/httpd/conf.d/ratelimit.conf`文件：

```
# 例如，在/download目录下限制带宽为500KB/秒
<IfModule mod_ratelimit.c>
    <Location /download>
        SetOutputFilter RATE_LIMIT
        SetEnv rate-limit 500
    </Location>
</IfModule>
```

`systemctl restart httpd`

访问位置以确保设置有效（上面是从限制的目录下载，下面是没有限制的）：

![httpd-mod-ratelimit](../Contents/httpd-mod-ratelimit.png)

