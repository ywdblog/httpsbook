1：7.3.1 小节

安装 certbot-bot：

```
# 下载 certbot-auto 客户端 
$ wget https://dl.eff.org/certbot-auto

# 修改程序权限
$ chmod a+x ./certbot-auto

# 查看帮助
$ ./certbot-auto --help 

# 查看 Certbot 客户端版本
$ ./certbot-auto --version
```

2：7.3.2 小节

```
# 注册帐号
$ certbot-auto register --agree-tos

# 更改帐号邮箱地址
$ certbot-auto register --update-registration --email admin@example.com
```

3：7.3.4 小节

```
# 使用非交互式操作
$ certbot-auto -n

# 测试证书
$ certbot-auto --test-cert 

# 模拟运行
$ certbot-auto --dry-run

# 运行 nginx 插件
$ certbot-auto run  --nginx 
```

4：7.3.5 小节

```
# apache 插件
$ certbot-auto run  --apache 
```

5：7.3.6 小节

```
# webroot 插件
$certbot-auto certonly --webroot \ 
   -w /usr/share/nginx/html -d www2.example.com \ 
   -w /usr/share/nginx/html -d www1.example.com \
   --rsa-key-size 2048 --dry-run
```

6：7.3.7 小节

```
# 默认使用 tls-sni 域名验证方式 
$ certbot-auto certonly --standalone -d www1.example.com  

# 使用 http 域名验证方式
$ certbot-auto certonly --standalone -d www1.example.com --preferred-challenges http
```

7：7.3.8 小节

```
# manual 插件
$ certbot-auto certonly  -d www4.example.com --manual --preferred-challenges dns
```

8：7.3.9 小节

更新证书：

```
$ certbot-auto certonly --standalone -d www4.example.com \
   --force-renewa --rsa-key-size 2048
```

扩展证书：

```
$ certbot-auto certonly --standalone --expand \
   -d www4.example.com -d www5.example.com  
```

9：7.3.10 小节

查看系统安装的证书：

```
$ certbot-auto certificates 
```

指定证书更新：

```
certbot-auto certonly --standalone --cert-name www4.example.com \
   -d www4.example.com -d www5.example.com  --expand --dry-run 
```

10：7.3.11 小节

撤销证书：

```
$ certbot-auto revoke --reason keycompromise \
   --cert-path /etc/letsencrypt/archive/www4.example.com/cert1.pem  
```

删除本地证书：

```
$ certbot delete --cert-name  www4.example.com
```

11：7.3.12 小节

```
# 扫描所有的证书文件，校验是否续期
$ certbot-auto renew 
 
# 扫描证书名是 www4.example.com 下的所有证书文件，校验是否续期
$ certbot-auto renew --cert-name www4.example.com 

# 校验某个证书是否续期
$ certbot-auto renew --cert-path /etc/letsencrypt/archive/www4.example.com/cert1.pem
$ certbot renew --post-hook "service nginx restart"
```

cron:

```
$ vim /etc/crontab 

$ 43 6 * * * certbot renew --post-hook "service nginx restart"
```

12： 7.3.13 小节

（1）基于 CSR 申请证书

```
# 生成密钥对和 CSR 文件 
$ openssl req  -new -sha256 -newkey rsa:2048 -nodes   \
  -subj '/CN=www6.example.com/O=Test, Inc./C=CN/ST=Beijing/L=Haidian' \
  -keyout www6.example.com_key.pem -out www6.example.com_csr.der 

# certbot 直接发送 CSR 文件给 Let's Encrypt
$ certbot-auto certonly --standalone \
   --preferred-challenges tls-sni-01 \
   --csr=www6.example.com_csr.der
```

（2）申请 ECDSA 证书

``` 
# 生成一个 ECC 密钥对
$ openssl ecparam -genkey -name secp384r1 -noout -out key.pem 

# 基于 ECC 密钥对生成 CSR 
$ openssl req -new -sha512 -nodes \
  -subj '/CN=www1.www.example.com/O=Test, Inc./C=CN/ST=Beijing/L=Haidian' \
  -key key.pem  -outform der -out www1.www.example.com_csr.der

# 使用 standalone 插件生成证书
$ certbot-auto certonly --standalone \
  --preferred-challenges tls-sni-01 \
  --csr=www1.www.example.com_csr.der
```