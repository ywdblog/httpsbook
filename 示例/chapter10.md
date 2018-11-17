1：10.1.1 小节

Modern 配置：

```
ssl_protocols TLSv1.2;

ssl_ciphers 'ECDHE-ECDSA-AES256-GCM-SHA384:
ECDHE-RSA-AES256-GCM-SHA384:
ECDHE-ECDSA-CHACHA20-POLY1305:
ECDHE-RSA-CHACHA20-POLY1305:
ECDHE-ECDSA-AES128-GCM-SHA256:
ECDHE-RSA-AES128-GCM-SHA256:
ECDHE-ECDSA-AES256-SHA384:
ECDHE-RSA-AES256-SHA384:
ECDHE-ECDSA-AES128-SHA256:
ECDHE-RSA-AES128-SHA256';
```

Intermediate 配置：

```
ssl_protocols TLSv1 TLSv1.1 TLSv1.2;

ssl_ciphers 'ECDHE-ECDSA-CHACHA20-POLY1305:
ECDHE-RSA-CHACHA20-POLY1305:
ECDHE-ECDSA-AES128-GCM-SHA256:
ECDHE-RSA-AES128-GCM-SHA256:
ECDHE-ECDSA-AES256-GCM-SHA384:
ECDHE-RSA-AES256-GCM-SHA384:
DHE-RSA-AES128-GCM-SHA256:
DHE-RSA-AES256-GCM-SHA384:
ECDHE-ECDSA-AES128-SHA256:
ECDHE-RSA-AES128-SHA256:
ECDHE-ECDSA-AES128-SHA:
ECDHE-RSA-AES256-SHA384:
ECDHE-RSA-AES128-SHA:
ECDHE-ECDSA-AES256-SHA384:
ECDHE-ECDSA-AES256-SHA:
ECDHE-RSA-AES256-SHA:
DHE-RSA-AES128-SHA256:
DHE-RSA-AES128-SHA:
DHE-RSA-AES256-SHA256:
DHE-RSA-AES256-SHA:
ECDHE-ECDSA-DES-CBC3-SHA:
ECDHE-RSA-DES-CBC3-SHA:
EDH-RSA-DES-CBC3-SHA:
AES128-GCM-SHA256:
AES256-GCM-SHA384:
AES128-SHA256:
AES256-SHA256:
AES128-SHA:
AES256-SHA:
DES-CBC3-SHA:
!DSS';
```

Old 配置：

```
ssl_protocols SSLv3 TLSv1 TLSv1.1 TLSv1.2;

ssl_ciphers 'ECDHE-ECDSA-CHACHA20-POLY1305:
ECDHE-RSA-CHACHA20-POLY1305:
ECDHE-RSA-AES128-GCM-SHA256:
ECDHE-ECDSA-AES128-GCM-SHA256:
ECDHE-RSA-AES256-GCM-SHA384:
ECDHE-ECDSA-AES256-GCM-SHA384:
DHE-RSA-AES128-GCM-SHA256:
DHE-DSS-AES128-GCM-SHA256:
kEDH+AESGCM:
ECDHE-RSA-AES128-SHA256:
ECDHE-ECDSA-AES128-SHA256:
ECDHE-RSA-AES128-SHA:
ECDHE-ECDSA-AES128-SHA:
ECDHE-RSA-AES256-SHA384:
ECDHE-ECDSA-AES256-SHA384:
ECDHE-RSA-AES256-SHA:
ECDHE-ECDSA-AES256-SHA:
DHE-RSA-AES128-SHA256:
DHE-RSA-AES128-SHA:
DHE-DSS-AES128-SHA256:
DHE-RSA-AES256-SHA256:
DHE-DSS-AES256-SHA:
DHE-RSA-AES256-SHA:
ECDHE-RSA-DES-CBC3-SHA:
ECDHE-ECDSA-DES-CBC3-SHA:
EDH-RSA-DES-CBC3-SHA:
AES128-GCM-SHA256:
AES256-GCM-SHA384:AES128-SHA256:
AES256-SHA256:
AES128-SHA:
AES256-SHA:
AES:
DES-CBC3-SHA:
HIGH:SEED:!aNULL:!eNULL:!EXPORT:
!DES:!RC4:!MD5:!PSK:!RSAPSK:
!aDH:!aECDH:!EDH-DSS-DES-CBC3-SHA:
!KRB5-DES-CBC3-SHA:!SRP';
```

nginx配置：

```
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    # Redirect all HTTP requests to HTTPS with a 301 Moved Permanently response.
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;

    # certs sent to the client in SERVER HELLO are concatenated in ssl_certificate
    ssl_certificate /path/to/signed_cert_plus_intermediates;
    ssl_certificate_key /path/to/private_key;
    ssl_session_timeout 1d;
    ssl_session_cache shared:SSL:50m;
    ssl_session_tickets off;

    # modern configuration. tweak to your needs.
    ssl_protocols TLSv1.2;
    ssl_ciphers 'ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256';
    ssl_prefer_server_ciphers on;

    # HSTS (ngx_http_headers_module is required) (15768000 seconds = 6 months)
    add_header Strict-Transport-Security max-age=15768000;

    # OCSP Stapling ---
    # fetch OCSP records from URL in ssl_certificate and cache them
    ssl_stapling on;
    ssl_stapling_verify on;

    ## verify chain of trust of OCSP response using Root CA and Intermediate certs
    ssl_trusted_certificate /path/to/root_CA_cert_plus_intermediates;

    resolver <IP DNS resolver>;

    ....
}
```

2：10.1.2 小节

```
ssl_protocols               TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
ssl_ecdh_curve              X25519:P-256:P-384:P-224:P-521;

ssl_ciphers                 '[ECDHE-ECDSA-AES128-GCM-SHA256|ECDHE-ECDSA-CHACHA20-POLY1305|ECDHE-RSA-AES128-GCM-SHA256|ECDHE-RSA-CHACHA20-POLY1305]:
ECDHE+AES128:
RSA+AES128:
ECDHE+AES256:
RSA+AES256:
ECDHE+3DES:
RSA+3DES';

ssl_prefer_server_ciphers   on;
```

```
ssl_prefer_server_ciphers  off;
ssl_ciphers  'ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-CHACHA20-POLY1305' ;
```

等价加密算法组：

```
ssl_prefer_server_ciphers   on;
ssl_ciphers  '[ECDHE-RSA-AES128-GCM-SHA256|ECDHE-RSA-CHACHA20-POLY1305]' ;
```

sslconfig 配置：

```
ssl_protocols               TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
ssl_ecdh_curve              X25519:P-256:P-384:P-224:P-521;

ssl_ciphers                 '[ECDHE-ECDSA-AES128-GCM-SHA256|ECDHE-ECDSA-CHACHA20-POLY1305|ECDHE-RSA-AES128-GCM-SHA256|ECDHE-RSA-CHACHA20-POLY1305]:
ECDHE+AES128:
RSA+AES128:
ECDHE+AES256:
RSA+AES256:
ECDHE+3DES:
RSA+3DES';

ssl_prefer_server_ciphers   on;
```

3：10.3.1 小节

（1）s_client

```
$ man s_client

$ openssl s_client -connect www.example.com:443

获取服务器证书并查看：

```
$ openssl s_client -connect www.example.com:443 2>&1 | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' >cert.pem

$ openssl x509 -text -in cert.pem  -noout
```

测试是否支持 smtp：

```
$ openssl s_client -connect smtp.sina.net:25 -starttls smtp
```

测试是否支持特定协议版本：

```
$ openssl s_client -connect www.example.com:443  -ssl3
```

指定本地根证书地址：

```
$ openssl s_client -connect www.example.com:443  -CApath /etc/ssl/certs
```

指定本地 CA 根证书：

```
$ openssl s_client -connect www.example.com:443  -CAfile /etc/ssl/certs/ca-certificates.crt

$ openssl s_client -connect www.example.com:443  -CAfile /usr/share/ca-certificates/mozilla/DigiCert_Global_Root_CA.crt

$ openssl s_client -connect www.example.com:443 -no-CAfile
```

查看状态：

```
$ openssl s_client -connect www.example.com:443 -state
```

SNI:

```
$ openssl s_client -connect www.example.com:443  -servername www.example.com
```

测试：

```
$ openssl s_client -connect www.example.com:443 -tlsextdebug
```

reconnect：

```
$ openssl s_client -connect www.example.com:443  2>&1  -reconnect  | grep "New\|Reuse"
```

debug：

```
$ openssl s_client -connect www.example.com:443  2>&1 -debug

$ openssl s_client -connect www.example.com:443  2>&1 -msg -msgfile https.cap
```

查看 ecc 命名曲线：

```
$ openssl ecparam -list_curves
```

（2）s_server

启动 443 HTTPS 网站：

```
$ openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem -days 365 -nodes

$ openssl s_server -key key.pem -cert cert.pem -accept localhost:4433 -www -WWW

$ openssl s_server -key key.pem -cert cert.pem -accept localhost:4433  \
  -cipher "ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305"
```

4：10.3.3 小节

（1）O-Saft

```
# 了解简短的使用说明
$ ./o-saft.pl -h

# 了解详细的使用说明
$ ./o-saft.pl --help
```

通过几个例子描述使用：

```
# 显示网站证书
$ ./o-saft.pl +certificate www.example.com

# 显示本地支持的所有密码套件
$ ./o-saft.pl +list

# 仅仅显示网站支持的密码套件
$ ./o-saft.pl +cipher --enabled www.example.com  

# 测试网站是否支持特定的密码套件
$ ./o-saft.pl +cipher --cipher=ADH-AES256-SHA www.example.com

# 对网站的握手进行调试
$ ./o-saft.pl +info www.example.com --trace

# 显示证书链信息
$ ./o-saft.pl www.example.com +chain_verify +verify +error_verify +chain
```

（2）RFC 5077 工具

安装该工具：

```
$ git clone https://github.com/vincentbernat/rfc5077.git
$ cd rfc5077/
$ git submodule init
$ git submodule update
$ make
```

运行 rfc5077-client 工具，对网站会话恢复进行测试：

```
$ ./rfc5077-client -s www.example.com 139.129.23.162
```

5：10.4.1 小节

（1）nginx 安装 

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

# 详细了解配置参数
$ ./configure --help

$ ./configure \
    --prefix=/usr/local/nginx1.13 \
    --with-pcre=../pcre-8.41 \
    --with-zlib=../zlib-1.2.11 \
    --with-http_ssl_module \
    --with-stream \
    --with-openssl=../openssl-1.1.0f \
    --with-openssl-opt="enable-ec_nistp_64_gcc_128"
 
$ make
$ make install
$ make clean
 
# Nginx 的配置文件
/usr/local/nginx1.13/conf/nginx.conf

# Nginx 二进制运行文件
/usr/local/nginx1.13/sbin/nginx
 
# 测试配置是否正确
$ /usr/local/nginx1.13/sbin/nginx -t

# 启动 Nginx
$ /usr/local/nginx1.13/sbin/nginx

$ nginx -V
```

（2）基本 https 指令配置

```
http {
   include       mime.types;
   default_type  application/octet-stream;

   log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                   '$status $body_bytes_sent "$http_referer" '
                   '"$http_user_agent" "$http_x_forwarded_for"';

   access_log  logs/access.log  main;

   sendfile        on;

   # HTTPS server
   server {
      listen  443 ssl;
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
