安装并配置DNS for CentOS7
===
---


#### 安装
```
# yum install -y bind bind-util
```

#### 配置

1. 使用原来的主机名或重新设定主机名
- 重新设定主机名
```
# hostnamectl set-hostname dns.zlx.com
```

- 检查设定的主机名
```
# hostname
```
2. 配置`/etc/named.com`
```
//
// named.conf
//
// Provided by Red Hat bind package to configure the ISC BIND named(8) DNS
// server as a caching only nameserver (as a localhost DNS resolver only).
//
// See /usr/share/doc/bind*/sample/ for example named configuration files.
//

options {
	listen-on port 53 { 10.70.77.37; 127.0.0.1; };
//	listen-on-v6 port 53 { ::1; };
	directory 	"/var/named";
	dump-file 	"/var/named/data/cache_dump.db";
	statistics-file "/var/named/data/named_stats.txt";
	memstatistics-file "/var/named/data/named_mem_stats.txt";
	allow-query     { localhost; };

	/* 
	 - If you are building an AUTHORITATIVE DNS server, do NOT enable recursion.
	 - If you are building a RECURSIVE (caching) DNS server, you need to enable 
	   recursion. 
	 - If your recursive DNS server has a public IP address, you MUST enable access 
	   control to limit queries to your legitimate users. Failing to do so will
	   cause your server to become part of large scale DNS amplification 
	   attacks. Implementing BCP38 within your network would greatly
	   reduce such attack surface 
	*/
	recursion yes;

	dnssec-enable yes;
	dnssec-validation yes;

	/* Path to ISC DLV key */
	bindkeys-file "/etc/named.iscdlv.key";

	managed-keys-directory "/var/named/dynamic";

	pid-file "/run/named/named.pid";
	session-keyfile "/run/named/session.key";
};

logging {
        channel default_debug {
                file "data/named.run";
                severity dynamic;
        };
};

zone "." IN {
	type hint;
	file "named.ca";
};

zone "zlx.com" IN {
    type master;
    file "/var/named/dynamic/zlx.com.zone";
    allow-update { none; };
};

include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";

```

3. 创建`/var/named/dynamic/zlx.com.zone`
```
$TTL 3H
@       IN SOA  dns.zlx.com. rname.zlx.com. (
                                      0       ; serial
                                      1D      ; refresh
                                      1H      ; retry
                                      1W      ; expire
                                      3H )    ; minimum
      NS              dns.zlx.com.
dns   IN       A       10.70.77.37
*      IN       A       10.70.77.37
```
`10.70.77.37`是DNS服务器的IP

4. 检查`named`配置并启动`named`
```
# named-checkconf
# systemctl start named
```

5. 验证
```
# nslookup www.zlx.com

```
输出：
```
Server:		127.0.0.1
Address:	127.0.0.1#53

Name:	www.zlx.com
Address: 10.70.77.37
```


更新log

- 本地DNS 2016.5.5
