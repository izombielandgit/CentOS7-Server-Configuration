### 5.1.22. 配置mod_security

使用mod_security模块配置Web应用程序防火墙（WAF）。

`yum -y install mod_security`

安装后，配置文件放在下面的目录中，并且设置被启用。其中有一些设置，还可以添加自己的规则：

`cat /etc/httpd/conf.d/mod_security.conf`

```
<IfModule mod_security2.c>
    # ModSecurity Core Rules Set configuration
        IncludeOptional modsecurity.d/*.conf
        IncludeOptional modsecurity.d/activated_rules/*.conf

    # Default recommended configuration
    SecRuleEngine On
    SecRequestBodyAccess On
    SecRule REQUEST_HEADERS:Content-Type "text/xml" \
         "id:'200000',phase:1,t:none,t:lowercase,pass,nolog,ctl:requestBodyProcessor=XML"

.....
.....
# 如果不希望在匹配规则时阻止请求，指定对参数“SecRuleEngine DetectionOnly”的更改
```

可以如下编写规则：

`SecRule VARIABLES OPERATOR [ACTIONS]`

每个参数有许多种值，请参考[官方文档](https://github.com/SpiderLabs/ModSecurity/wiki/Reference-Manual)。

举例设置一些规则并验证它正常工作：

编辑`/etc/httpd/modsecurity.d/activated_rules/rules-01.conf`文件：

```
# 匹配规则时的默认操作
SecDefaultAction "phase:2,deny,log,status:406"

# “etc/passwd”包含在请求URI中
SecRule REQUEST_URI "etc/passwd" "id:'500001'"

# “../”包括在请求URI中
SecRule REQUEST_URI "\.\./" "id:'500002'"

# “<SCRIPT”包含在参数中
SecRule ARGS "<[Ss][Cc][Rr][Ii][Pp][Tt]" "id:'500003'"

# “SELECT FROM”包含在参数中
SecRule ARGS "[Ss][Ee][Ll][Ee][Cc][Tt][[:space:]]+[Ff][Rr][Oo][Mm]" "id:'500004'"
```

`systemctl restart httpd`

访问包含您设置的字词的URI，并验证其是否正常工作：

![httpd-mod-security](../Contents/httpd-mod-security.png)

mod_security的日志放在如下所示的目录中：

`cat /var/log/httpd/modsec_audit.log`

```
--75d36531-A--
[28/Oct/2015:13:52:52 +0900] VjBUpAKZ9yAFgyhKj8zyyAAAAAE 10.0.0.108 53545 10.0.0.31 80
--75d36531-B--
GET /?../../etc/passwd HTTP/1.1
Host: www.srv.world
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:38.0) Gecko/20100101 Firefox/38.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: keep-alive

--75d36531-F--
HTTP/1.1 406 Not Acceptable
Content-Length: 251
Keep-Alive: timeout=5, max=100
Connection: Keep-Alive
Content-Type: text/html; charset=iso-8859-1

--75d36531-E--

--75d36531-H--
Message: Access denied with code 406 (phase 2). Pattern match "etc/passwd" 
at REQUEST_URI. [file "/etc/httpd/modsecurity.d/activated_rules/rules-01.conf"] [line "3"] [id "500001"]
Action: Intercepted (phase 2)
Stopwatch: 1446007972909468 1333 (- - -)
Stopwatch2: 1446007972909468 1333; combined=418, p1=395, p2=17, p3=0, p4=0, p5=6, sr=116, sw=0, l=0, gc=0
Response-Body-Transformed: Dechunked
Producer: ModSecurity for Apache/2.7.3 (http://www.modsecurity.org/); OWASP_CRS/2.2.6.
Server: Apache/2.4.6 (CentOS)
Engine-Mode: "ENABLED"

--75d36531-Z--
```

普通规则由官方存储库提供，很容易应用。但可能需要自定义它们让自己的网站不阻止必要的请求：

`yum -y install mod_security_crs`

规则放置如下，它们被链接到目录`/etc/httpd/modsecurity.d/activated_rules`：

`ll /usr/lib/modsecurity.d/base_rules`

```
total 332
-rw-r--r-- 1 root root  1980 Jun 10  2014 modsecurity_35_bad_robots.data
-rw-r--r-- 1 root root   386 Jun 10  2014 modsecurity_35_scanners.data
-rw-r--r-- 1 root root  3928 Jun 10  2014 modsecurity_40_generic_attacks.data
-rw-r--r-- 1 root root  2610 Jun 10  2014 modsecurity_41_sql_injection_attacks.data
-rw-r--r-- 1 root root  2224 Jun 10  2014 modsecurity_50_outbound.data
-rw-r--r-- 1 root root 56714 Jun 10  2014 modsecurity_50_outbound_malware.data
-rw-r--r-- 1 root root 22861 Jun 10  2014 modsecurity_crs_20_protocol_violations.conf
-rw-r--r-- 1 root root  6915 Jun 10  2014 modsecurity_crs_21_protocol_anomalies.conf
-rw-r--r-- 1 root root  3792 Jun 10  2014 modsecurity_crs_23_request_limits.conf
-rw-r--r-- 1 root root  6933 Jun 10  2014 modsecurity_crs_30_http_policy.conf
-rw-r--r-- 1 root root  5394 Jun 10  2014 modsecurity_crs_35_bad_robots.conf
-rw-r--r-- 1 root root 19157 Jun 10  2014 modsecurity_crs_40_generic_attacks.conf
-rw-r--r-- 1 root root 43961 Jun 10  2014 modsecurity_crs_41_sql_injection_attacks.conf
-rw-r--r-- 1 root root 87470 Jun 10  2014 modsecurity_crs_41_xss_attacks.conf
-rw-r--r-- 1 root root  1795 Jun 10  2014 modsecurity_crs_42_tight_security.conf
-rw-r--r-- 1 root root  3660 Jun 10  2014 modsecurity_crs_45_trojans.conf
-rw-r--r-- 1 root root  2253 Jun 10  2014 modsecurity_crs_47_common_exceptions.conf
-rw-r--r-- 1 root root  2787 Jun 10  2014 modsecurity_crs_48_local_exceptions.conf.example
-rw-r--r-- 1 root root  1835 Jun 10  2014 modsecurity_crs_49_inbound_blocking.conf
-rw-r--r-- 1 root root 22314 Jun 10  2014 modsecurity_crs_50_outbound.conf
-rw-r--r-- 1 root root  1448 Jun 10  2014 modsecurity_crs_59_outbound_blocking.conf
-rw-r--r-- 1 root root  2674 Jun 10  2014 modsecurity_crs_60_correlation.conf
```