### 5.1.20. 配置mod_limitipconn

使用mod_limitipconn限制每个IP地址的并发连接。

`yum --enablerepo=epel -y install mod_limitipconn` # 从EPEL安装

编辑`/etc/httpd/conf.d/limitipconn.conf`文件：

```
# 默认设置没有限制
MaxConnPerIP 0

# /limit目录设置
<Location /limit>
    # 限制并发连接3
    MaxConnPerIP 3
    # 如果MIME类型为“text/*”，则不适用上面规则
    NoIPLimit text/*
</Location>

# /limit2目录设置
<Location /limit2>
    # 限制并发连接2
    MaxConnPerIP 2
    # 如果MIME类型为“application/x-tar”，则不适用上面规则
    OnlyIPLimit application/x-tar
</Location>
```

`systemctl restart httpd`

使用httpd-tools包中包含的命令“ab”验证是否正常工作，如下所示：

`ab -n 10 -c 10 http://localhost/limit/index.html`

```
This is ApacheBench, Version 2.3 <$Revision: 1430300 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking localhost (be patient).....done

Server Software:        Apache/2.4.6
Server Hostname:        localhost
Server Port:            80

Document Path:          /limit/index.html
Document Length:        130 bytes

Concurrency Level:      10
Time taken for tests:   0.004 seconds
Complete requests:      10
Failed requests:        0
Write errors:           0
Total transferred:      3910 bytes
HTML transferred:       1300 bytes
Requests per second:    2223.21 [#/sec] (mean)
Time per request:       4.498 [ms] (mean)
Time per request:       0.450 [ms] (mean, across all concurrent requests)
Transfer rate:          848.90 [Kbytes/sec] received
.....
.....
```

`ab -n 10 -c 10 http://localhost/limit/test.gif`

```
This is ApacheBench, Version 2.3 <$Revision: 1430300 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking localhost (be patient).....done


Server Software:        Apache/2.4.6
Server Hostname:        localhost
Server Port:            80

Document Path:          /limit/test.gif
Document Length:        228 bytes

Concurrency Level:      10
Time taken for tests:   0.005 seconds
Complete requests:      10
Failed requests:        7
   (Connect: 0, Receive: 0, Length: 7, Exceptions: 0)
Write errors:           0
Non-2xx responses:      7
Total transferred:      4838 bytes
HTML transferred:       2777 bytes
Requests per second:    2182.45 [#/sec] (mean)
Time per request:       4.582 [ms] (mean)
Time per request:       0.458 [ms] (mean, across all concurrent requests)
Transfer rate:          1031.12 [Kbytes/sec] received
.....
.....
```

`ab -n 10 -c 10 http://localhost/limit2/test.tar`

```
This is ApacheBench, Version 2.3 <$Revision: 1430300 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking localhost (be patient).....done


Server Software:        Apache/2.4.6
Server Hostname:        localhost
Server Port:            80

Document Path:          /limit2/test.tar
Document Length:        10240 bytes

Concurrency Level:      10
Time taken for tests:   0.006 seconds
Complete requests:      10
Failed requests:        8
   (Connect: 0, Receive: 0, Length: 8, Exceptions: 0)
Write errors:           0
Non-2xx responses:      8
Total transferred:      24900 bytes
HTML transferred:       22872 bytes
Requests per second:    1785.40 [#/sec] (mean)
Time per request:       5.601 [ms] (mean)
Time per request:       0.560 [ms] (mean, across all concurrent requests)
Transfer rate:          4341.44 [Kbytes/sec] received
.....
.....
```