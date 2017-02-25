### 5.1.16 Python + mod_wsgi

安装mod_wsgi（WSGI：Web Server Gateway Interface/Web服务器网关接口），使Python脚本更快。

`yum -y install mod_wsgi` # 安装mod_wsgi

编辑`/etc/httpd/conf.d/wsgi.conf`，配置mod_wsgi：

示例为让可以访问`/test_wsgi`的后端是`/var/www/html/test_wsgi.py`：

```
WSGIScriptAlias /test_wsgi /var/www/html/test_wsgi.py
```

`systemctl restart httpd`

创建在上面设置的测试脚本：

编辑`/var/www/html/test_wsgi.py`文件：

```
def application(environ,start_response):
    status = '200 OK'
    html = '<html>\n' \
           '<body>\n' \
           '<div style="width: 100%; font-size: 40px; font-weight: bold; text-align: center;">\n' \
           'mod_wsgi Test Page\n' \
           '</div>\n' \
           '</body>\n' \
           '</html>\n'
    response_header = [('Content-type','text/html')]
    start_response(status,response_header)
    return [html]
```

![httpd-mod-wsgi-test-page](../Contents/httpd-mod-wsgi-test-page.png)

如果[使用Django](https://www.server-world.info/en/note?os=CentOS_7&p=django)

编辑`/etc/httpd/conf.d/django.conf`文件，配置：

示例为在“cent”用户的`/home/cent/venv/testproject`下配置“testapp”：

```
WSGIDaemonProcess testapp python-path=/home/cent/venv/testproject:/home/cent/venv/lib/python2.7/site-packages
WSGIProcessGroup testapp
WSGIScriptAlias /django /home/cent/venv/testproject/testproject/wsgi.py

<Directory /home/cent/venv/testproject>
    Require all granted
</Directory>
```

`systemctl restart httpd`

![httpd-django-test-page](../Contents/httpd-django-test-page.png)