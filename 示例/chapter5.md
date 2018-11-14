1：5.2.1 小节

生成私钥对和 CSR：

```
$ openssl req \
       -newkey rsa:1024 -nodes -keyout example_key.pem \
       -out example_csr.pem
```

生成自签名证书：

```
$ openssl x509 \
       -signkey example_key.pem \
       -in example_csr.pem \
       -req -days 365 -out example_cert.pem 
```

2：5.2.3 小节

```
# 下载 Certbot 客户端 
$ git clone https://github.com/certbot/certbot

$ cd certbot

# 一步生成证书
$./certbot-auto certonly --webroot -w /usr/nginx/web -d rss.newyingyong.cn
```

3：5.3.1 小节

安装 Nginx： 

```
$ apt-get install nginx 

# 显示 Nginx 版本
$ nginx -V
```

配置 HTTPS：

```
$ vim  /etc/nginx/sites-enabled/default

server {
     # 启用 443 端口 
     listen 443 ssl;

     # 服务器域名 
     server_name www.example.com;

     # 程序目录
     root /var/www/html;

     # 路径可以自定义
     # ssl_certificate 表示证书路径
     ssl_certificate /etc/cert/fullchain.pem;

     # ssl_certificate_key 表示密钥对路径
     ssl_certificate_key /etc/cert/privkey.pem;
}
```

重新启动 Nginx：

```
$ service nginx restart
```

4：5.3.2 小节 

安装：

```
$ apt-get install apache2

# 启用 ssl 模块
$ a2enmod ssl 

# 显示版本
$ apache2 -v  
```

配置 HTTPS：

```
# 编辑配置文件
vim /etc/apache2/sites-available/default-ssl.conf

<VirtualHost _default_:443>
    DocumentRoot /var/www/html
    ServerName www.example.com 

    # 开启 SSL 
    SSLEngine on
    SSLCertificateFile /etc/cert/fullchain.pem;
    SSLCertificateKeyFile /etc/cert/privkey.pem;
</VirtualHost>
```

重新启动 Apache2：

```
$ a2ensite default-ssl.conf 
$ service apache2 restart
```

5：5.4 小节

301 首页重定向:

```
server {
    listen       80;
    server_name  www.example.com;
    rewrite      ^ https://$server_name;
}
```

HTTP URL to HTTPS URL：

```
server {
    listen       80;
    server_name  www.example.com;
    rewrite      ^ https://$server_name$request_uri? permanent;
}
```

（6）5.5.2 小节

HSTS 配置：

```
server {
    listen       80;
    server_name  www.example.com;
    rewrite       ^ https://$server_name$request_uri? permanent;
}

server {
    listen       443 ssl;
    server_name  www.example.com;

    # 有效期是 1000 秒、域名下的子域名都配置 HSTS 
    add_header Strict-Transport-Security 'max-age=1000; includeSubDomains; preload;'

    ssl_certificate /etc/cert/fullchain.pem;
    ssl_certificate_key /etc/cert/privkey.pem;
```

