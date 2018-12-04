1：9.1 小节

```
# 查看本机openssl是否支持RC4算法
$ openssl ciphers -V 'RC4' | column -t   

# 查看本机openssl是否支持某个密码套件
$ openssl ciphers -V 'ECDHE-ECDSA-AES256-GCM-SHA384' | column -t
```

2: 9.1.2 小节

身份验证和密钥交换关键字：

```
$ openssl ciphers -V "aRSA" |  column -t 

$ openssl ciphers -V "kRSA" |  column -t

$ openssl ciphers -V "ECDSA" |  column -t    

$ openssl ciphers -V "DH" |  column -t 

$ openssl ciphers -V "EDH" |  column -t

$ openssl ciphers -V "ADH" |  column -t  

$ openssl ciphers -V 'kECDHE' | column -t  

$ openssl ciphers -V 'ECDHE' | column -t

$ openssl ciphers -V 'DSS' | column -t  

$ openssl ciphers -V 'kPSK' | column -t   

$ openssl ciphers -V 'kECDHEPSK' | column -t 

$ ciphers -V 'kDHEPSK' | column -t   

$ openssl ciphers -V 'kRSAPSK' | column -t  

$ openssl ciphers -V 'SRP' | column -t 
```

加密算法相关密码套件：

```
$ openssl ciphers -V 'AESGCM' | column -t  

$ openssl ciphers -V 'CHACHA20' | column -t  

$ openssl ciphers -V 'AESCCM' | column -t

$ openssl ciphers -V 'AES' | column -t 

$ openssl ciphers -V '3DES' | column -t
```

摘要算法关键字：

```
$ openssl ciphers -V 'MD5' | column -t 

$ openssl ciphers -V 'SHA' | column -t  

$ openssl ciphers -V 'SHA384' | column -t 
```

分组关键字：

```
$ openssl ciphers -V 'COMPLEMENTOFALL' | column -t  

$ openssl ciphers -V 'HIGH' | column -t  

$ openssl ciphers -V 'MEDIUM' | column -t

$ openssl ciphers -V 'LOW' | column -t

$ openssl ciphers -V 'TLSv1.0' | column -t 

$ openssl ciphers -V 'AES+SHA256' | column -t  

$ openssl ciphers -V 'AES:SHA256' | column -t  

$ openssl ciphers -V 'eNULL:SHA384' | column -t

$ openssl ciphers -V 'DEFAULT:!CHACHA20:CHACHA20' | column -t 

$ openssl ciphers -V 'DEFAULT:-AESGCM:AESGCM' | column -t 

$ openssl ciphers -V 'DEFAULT:+3DES' | column -t

$ openssl ciphers -V 'AES:+SHA256' | column -t   | wc -l                    

$ openssl ciphers -V 'AES+SHA256' | column -t   | wc -l

$ openssl ciphers -v 'AES:@STRENGTH'
```

3：9.2 小节

文档、工具地址如下：

| 网站 | 文档或工具名称|地址|说明|
| :-------------:|:-------------:|:-------------:|:-------------:|
|SSL Labs|  TLS Deployment Best Practices | https://www.ssllabs.com/projects/best-practices/index.html | 最佳部署文档，读者应该比较文档每次更新的内容|
| SSL Labs| SSL Server Test |  https://www.ssllabs.com/ssltest| 对服务器的 HTTPS 配置进行测试，指出潜在的问题，并对安全级别打分|
|SSL Labs| SSL Client Test | https://www.ssllabs.com/ssltest/viewMyClient.html | HTTPS 网站也会涉及到客户端，该工具可以测试客户端的配置情况 |
|SSL Labs| SSL Pulse | https://www.ssllabs.com/ssltest/viewMyClient.html | 对全球顶尖 HTTPS 网站进行长期跟踪，统计和 HTTPS 有关的一些数据|
| Mozilla | 最佳部署文档| https://wiki.mozilla.org/Security/Server_Side_TLS  | 重点介绍密码套件配置、HTTPS 潜在的攻击、最佳部署的一些指导|
|Mozilla | Mozilla SSL Configuration Generator| https://mozilla.github.io/server-side-tls/ssl-config-generator/| 用于自动化为服务器配置 HTTPS 协议|
| RFC |  Summarizing Known Attacks on TLS|https://tools.ietf.org/html/rfc7457 | 详细描述 TLS/SSL 协议历史上出现过的漏洞 |
| RFC | Recommendations for Secure Use of TLS | https://tools.ietf.org/html/rfc7525 | 描述如何更好的部署 HTTPS 网站|

4：9.3.1 小节

```
# 查看是否启用窗口缩放
$ sysctl net.ipv4.tcp_window_scaling

# 设置启用窗口缩放
$ sysctl -w net.ipv4.tcp_window_scaling=1 
```

查看初始拥塞窗口大小的方法如下： 

```
$ ss -nli|fgrep cwnd

# 输出
rto:1000 mss:536 cwnd:10
```

修改初始拥塞窗口大小
```
# 修改某个网卡的 cwnd 
ip route change 127.0.0.1 initcwnd 10 
```

控制满启动：

```
# 查看是否禁止慢启动重启
$ sysctl net.ipv4.tcp_slow_start_after_idle

# 设置慢启动重启
sysctl -w net.ipv4.tcp_slow_start_after_idle=0
```

5：9.3.3 小节

编译 openssl 和 nginx：
```
# 下载较新的 Nginx 版本
$ wget http://nginx.org/download/nginx-1.13.5.tar.gz

# 下载 OpenSSL 库
$ wget https://www.openssl.org/source/openssl-1.1.0f.tar.gz

$ wget http://zlib.net/zlib-1.2.11.tar.gz
$ wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.41.tar.gz

# 解压缩
$ tar xvf nginx-1.13.5.tar.gz
$ tar xvf openssl-1.1.0f.tar.gz 
$ tar xvf zlib-1.2.11.tar.gz
$ tar xvf pcre-8.41.tar.gz 

$ cd nginx-1.13.5
$ ./configure --help

$ ./configure \
--prefix=/usr/local/nginx1.13 \
--with-pcre=../pcre-8.41 \
--with-zlib=../zlib-1.2.11 \
--with-http_ssl_module \
--with-http_v2_module  \
--with-stream \
--with-openssl=../openssl-1.1.0f
$ make 
$ make install
$ make clean 
```

配置 HTTP/2 网站：

```
server {
  listen  443 ssl http2;
  server_name  www.example.com;

  ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
  ssl_certificate      cert.pem;
  ssl_certificate_key  cert.key;
  ssl_ciphers  HIGH:!aNULL:!MD5;

  location / {
     root   html;
     index  index.html index.htm;
  }
}
```

编译curl支持http/2：

```
$ apt-get install build-essential libnghttp2-dev libssl-dev
 
$ git clone https://github.com/tatsuhiro-t/nghttp2.git
$ cd nghttp2
$ autoreconf -i
$ automake
$ autoconf
$ ./configure
$ make
$ make install

$ wget https://curl.haxx.se/download/curl-7.56.0.tar.gz
$ tar -xvf curl-7.56.0.tar.gz

$ cd curl-7.58.0
$ ./configure --with-nghttp2 --prefix=/usr/local --with-ssl
$ make
$ make install

$ /usr/local/bin/curl -V
```

测试网站是否支持 HTTP/2：

```
$ curl  --http2 -k -I "https://www.example.com"  
```