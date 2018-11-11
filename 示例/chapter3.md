1：3.3.1 小节

```
# 显示机器支持的密码套件
$ openssl ciphers -V | column -t 

# 测试某个网站是否支持特定协议、特定密码套件的连接
$ openssl s_client -cipher "ECDHE-RSA-AES128-SHA" -connect www.example.com:443 -tls1_1 
```