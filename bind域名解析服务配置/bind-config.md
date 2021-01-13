# 1.安装bind包

```
yum install  bind  bind-libs bind-utils bind-chroot  -y
```

```
rpm -qa|grep bind|grep '^bind'
bind-chroot-9.11.20-5.el8.x86_64
bind-libs-9.11.20-5.el8.x86_64
bind-utils-9.11.20-5.el8.x86_64
bind-export-libs-9.11.20-5.el8.x86_64
bind-libs-lite-9.11.20-5.el8.x86_64
bind-9.11.20-5.el8.x86_64
bind-license-9.11.20-5.el8.noarch

```

# 2.配置/etc/named.conf文件,修改下面参数

```
listen-on port 53 { any; };
directory  "/var/named";
allow-query     { any; };
```



# 3.修改/etc/named.rfc1912.zones，加上正向解析和反向解析的文件

```
zone "yoyo.com" IN {
 type master;
 file "yoyo.com.zone";
};
zone "12.3.2.in-addr.arpa" IN {
 type master;
 file "12.3.2.zone";
};
```

# 4.配置解析文件

```
创建并配置正向解析文件/var/named/yoyo.com.zone
$TTL 1D
yoyo.com. IN SOA yoyo.com. root (0 1H 15M 1W 1D)
yoyo.com. IN NS dns1.yoyo.com.
dns1.yoyo.com. IN A 127.0.0.1
www.yoyo.com. IN A 2.3.12.13
```

```
创建并配置反向解析文件/var/named/12.3.2.zone
$TTL 1D
@ IN SOA 12.3.2.in-addr.arpa. root (0 1H 15M 1W 1D)
@ IN NS dns1.yoyo.com.
127.0.0.1 IN PTR dns1.yoyo.com.
13 IN PTR www.yoyo.com.
```

# 5.named-checkconf检查配置文件是否有误

```
 service named start #启动DNS服务
```

# 6.测试可以修改linux的DNS服务器指向安装的bind服务

```
1>setup图形化界面配置
2>/etc/sysconfig/network-scripts/ifcfg-eth0添加DNS1
3>/etc/resolv.conf 添加nameserver   生效顺序是ifcfg-eth0->resolv.conf文件
```

# 7.直接测试

```
正向解析：
[root@MiWiFi-R3-srv ~]# dig @127.0.0.1 www.yoyo.com

; <<>> DiG 9.11.20-RedHat-9.11.20-5.el8 <<>> @127.0.0.1 www.yoyo.com
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 31142
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 52abe40876799334aff4bb695ffef3a6b8e6c6143ca54982 (good)
;; QUESTION SECTION:
;www.yoyo.com.			IN	A

;; ANSWER SECTION:
www.yoyo.com.		86400	IN	A	2.3.12.13

;; AUTHORITY SECTION:
yoyo.com.		86400	IN	NS	dns1.yoyo.com.

;; ADDITIONAL SECTION:
dns1.yoyo.com.		86400	IN	A	127.0.0.1

;; Query time: 0 msec
;; SERVER: 127.0.0.1#53(127.0.0.1)
;; WHEN: 三 1月 13 21:20:38 CST 2021
;; MSG SIZE  rcvd: 120


反向解析：
[root@MiWiFi-R3-srv ~]#  dig @127.0.0.1 -x 2.3.12.13

; <<>> DiG 9.11.20-RedHat-9.11.20-5.el8 <<>> @127.0.0.1 -x 2.3.12.13
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 27509
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 9a10a64da8cc18472b74f6725ffef37ed8377ae26c703aa3 (good)
;; QUESTION SECTION:
;13.12.3.2.in-addr.arpa.		IN	PTR

;; ANSWER SECTION:
13.12.3.2.in-addr.arpa.	86400	IN	PTR	www.yoyo.com.

;; AUTHORITY SECTION:
12.3.2.in-addr.arpa.	86400	IN	NS	dns1.yoyo.com.

;; ADDITIONAL SECTION:
dns1.yoyo.com.		86400	IN	A	127.0.0.1

;; Query time: 0 msec
;; SERVER: 127.0.0.1#53(127.0.0.1)
;; WHEN: 三 1月 13 21:19:58 CST 2021
;; MSG SIZE  rcvd: 140

```

```
[root@MiWiFi-R3-srv ~]#  nslookup
> server 127.0.0.1
Default server: 127.0.0.1
Address: 127.0.0.1#53
> www.yoyo.com
Server:		127.0.0.1
Address:	127.0.0.1#53

Name:	www.yoyo.com
Address: 2.3.12.13
> 2.3.12.13
13.12.3.2.in-addr.arpa	name = www.yoyo.com.
```

