1：6.3.5 小节

获取 Let’s Encrypt ISRG 中间证书公钥：

```
# 下载 Let’s Encrypt ISRG 中间证书
$ wget https://letsencrypt.org/certs/letsencryptauthorityx3.pem.txt -O isrg.pem

# 显示证书包含的公钥摘要值，该公钥用来验证服务器实体证书的签名
$ openssl x509 -in isrg.pem -subject_hash -noout

# 输出值
4f06f81d
```

获取 Let’s Encrypt DST 中间证书公钥：

```
# 下载 Let’s Encrypt DST 中间证书
$ wget https://letsencrypt.org/certs/lets-encrypt-x3-cross-signed.pem.txt -O dst.pem

# 显示证书包含的公钥摘要值，该公钥用来验证服务器实体证书的签名
$ openssl x509 -in dst.pem -subject_hash -noout

# 输出值
4f06f81d
```

2：6.7.2 小节

（1）PKCS#12 格式

通过 OpenSSL pkcs12 子命令将密钥对（privkey.pem）、服务器实体证书（cert.pem）、中间证书（chain.pem）转换成一个文件：

```
$ openssl pkcs12 \
    -export -out cert.pfx \
    -inkey privkey.pem -in cert.pem -certfile chain.pem
```

从 cert.pfx 导出密钥对和证书：

```
# 导出密钥对
$ openssl pkcs12 -in cert.pfx  -nodes -nocerts -out new_privkey.pem

# 导出服务器实体证书
$ openssl pkcs12 -in cert.pfx  -nodes -clcerts -out new_cert.pem

# 导出中间证书
$ openssl pkcs12 -in cert.pfx  -nodes -cacerts -out new_chain.pem  
```

（2）PKCS#7 格式

生成 cert.p7b 文件：

```
$ openssl crl2pkcs7 -nocrl -certfile cert.pem -certfile chain.pem -out cert.p7b
```
从 cert.p7b 文件中导出服务器证书文件和中间证书文件：

```
# 导出完整证书链文件，服务器实体证书在文件顶部，中间证书在文件底部
$ openssl pkcs7 -print_certs -in cert.p7b -out fullchain.cer
```

3：6.7.3 小节

使用 s_client 获取线上证书：

```
$ openssl s_client -connect www.example.com:443  -showcerts  2>&1 </dev/null
```

使用 Shell 命令拆分服务器实体证书和中间证书：

```
$ openssl s_client -connect www.example.com:443 -showcerts  2>&1 </dev/null \
  | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p'  \
  > www_fullchain.pem

$ cat www_fullchain.pem  | awk 'split_after==1{n++;split_after=0} \
    /-----END CERTIFICATE-----/ {split_after=1} \
    {print > "www_cert" n ".pem"}'

# 重命名中间证书，在本列中中间证书只有一个.
$ mv www_cert1.pem www_chain.pem

# 查看生成的各个文件
$ tree
```

获取根证书：

```
$ openssl x509 -in  www_chain.pem -noout -text | grep "CA Issuers" 

#下载根证书文件
$ wget http://apps.identrust.com/roots/dstrootcax3.p7c -O DST_ROOT.p7c

# 转换成 PEM 格式
$ openssl pkcs7 -inform der -in DST_ROOT.p7c -print_certs -out DST_ROOT.pem

# 检查证书签发者
$ openssl x509 -in DST_ROOT.pem -issuer
```

4：6.7.4 小节

下载 Mozilla 根证书库：

```
$ wget https://raw.githubusercontent.com/curl/curl/master/lib/mk-ca-bundle.pl

$ chmod 0777 mk-ca-bundle.pl

$ ./mk-ca-bundle.pl
```

更新系统的证书库：

```
$ update-ca-certificates
```

自签名证书同步到系统根证书库中：

```
$ mkdir /usr/local/share/ca-certificates/extra

# 拷贝自签名证书，文件后缀 crt 
$ cp self-ertificate.crt /usr/local/share/ca-certificates/extra

$ update-ca-certificates
```

5：6.7.5 小节

生成 CSR：

```
$ openssl req \
    -new -sha256 -newkey rsa:2048 -nodes \
    -subj '/CN=www1.example.com,www2.example.com/O=Test, Inc./C=CN/ST=Beijing/L=Haidian' \
    -keyout example_key.pem -out example_csr.pem
```

查看 CSR：

```
$ openssl req -in example_csr.pem -noout -text
```

校验 CSR 签名：

```
$ openssl req -in myreq.pem -noout -verify -key example_csr.pem
```

CSR 格式转换：

```
# PEM 格式转换为 DER 格式
$ openssl req -in example_csr.pem -out example_csr.der -outform DER

# DER 格式转换为 PEM 格式
$ openssl req -in example_csr.der -inform DER -out example_csr.pem -outform PEM   
```

6：6.7.6 小节

生成自签名证书

```
$ echo "subjectAltName=DNS:www1.example.com,DNS:www1.example.com" >>certext.ext 

$ openssl x509 -req -days 365 -in example_csr.pem \
   -signkey example_key.pem -out example_cert.pem \
   -extfile certext.ext
```

7：6.7.7 小节 

证书内部结构：

```
# 查看证书完整信息
$ openssl x509 -text -in cert.pem -noout

# 查看证书包含的公钥
$ openssl x509 -in cert.pem -pubkey

# 查看那个 CA 机构签发了证书
$ openssl x509 -in cert.pem -issuer

# 查看证书的有效期
$ openssl x509 -in cert.pem -enddate
```

查看服务器实体证书：

```
$ openssl x509 -in cert.pem -text -noout
```

查看 Let's Encrypt 中间证书：

```
$ openssl x509 -in cert.pem -text -noout  
```

查看根证书：

```
$ openssl x509 -in  DST_ROOT.pem -noout -text   
```

8：6.7.8 小节

手动校验：

```
# 下载服务器实体证书
$ openssl s_client -connect www.sina.com.cn:443 2>&1 < /dev/null | sed -n '/-----BEGIN/,/-----END/p' > www_cert.pem

# 找到服务器实体证书的 CRL 分发点
$ openssl x509 -in www_cert.pem -noout -text | grep "crl"

# 输出 CRL 分发点地址
URI:http://gn.symcb.com/gn.crl

# 下载 CRLs 文件
$ wget "http://gn.symcb.com/gn.crl"

# 查看 CRLs 文件内容
$ openssl crl -inform DER -text -noout -in gn.crl  
```

自动校验：

```
# 下载证书链文件
$ openssl s_client -connect www.sina.com.cn:443 -showcerts  2>&1 </dev/null \
   | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p'    \
   > www_fullchain.pem

# 拆分证书文件，www_cert.pem 是服务器实体证书，www_chain1.pem 是中间证书
$ cat www_fullchain.pem  | awk 'split_after==1{n++;split_after=0} \
    /-----END CERTIFICATE-----/ {split_after=1} \
    {print > "www_cert" n ".pem"}'

# 重命名中间证书，在该例中，中间证书只有一个。
$ mv www_cert1.pem www_chain.pem

# 找到服务器实体证书的 CRL 分发点
$ openssl x509 -in www_cert.pem -noout -text | grep "crl"

# 输出 CRL 分发点地址
URI:http://gn.symcb.com/gn.crl

# 下载 CRLs 文件
$ wget "http://gn.symcb.com/gn.crl"

# 将 CRLs 文件转换为 PEM 格式
$ openssl crl -inform DER -in gn.crl -outform PEM -out crl.pem

# 合并中间证书和 CRLs 文件
$ cat www_chain.pem crl.pem > crl_chain.pem

# 校验，-CAfile 表示完整证书链
$ openssl verify -crl_check -CAfile crl_chain.pem www_cert.pem

# 输出 ok 表示服务器实体证书没有被吊销
www_cert.pem: OK
```

比较：

```
# 查询服务器实体证书的签发者
$ openssl x509 -in www_cert.pem -issuer -pubkey
# 输出
issuer=C = US, O = GeoTrust Inc., CN = GeoTrust SSL CA - G3

# 查询 CRLs 的签发者
$ openssl crl -inform DER  -noout -in gn.crl  -issuer   
# 输出
issuer=C = US, O = GeoTrust Inc., CN = GeoTrust SSL CA - G3
```

9：6.7.9 小节

（1）校验 Let's Encrypt 的 OCSP 服务

```
# 从服务器实体证书中获取 OCSP 的地址
$ openssl x509 -in cert.pem -noout -ocsp_uri

# 输出 OCSP URL 地址
http://ocsp.int-x3.letsencrypt.org

# 校验 OCSP  
$ openssl ocsp -issuer chain.pem -cert cert.pem -CAfile chain.pem \
   -no_nonce --text -url  http://ocsp.int-x3.letsencrypt.org \
   -header Host=ocsp.int-x3.letsencrypt.org
```

（2）校验 GeoTrust 的 OCSP 响应
 
```
$ openssl ocsp -issuer www_chain.pem -cert www_cert.pem  \
   -url  http://gn.symcd.com -CAfile www_chain.pem --text -no_nonce
```

10：6.7.10 小节

使用不支持 OCSP 封套的 HTTPS 网站进行演示：

```
$ openssl s_client -connect letsencrypt.org:443 -status -tlsextdebug < /dev/null 2>&1 \
   | grep -i "OCSP response"
```

使用支持 OCSP 封套的 HTTPS 网站进行演示

```
$ openssl s_client -connect www.baidu.com:443 -status -tlsextdebug < /dev/null 2>&1
```

11：6.8.2 小节

查看 github 的 SCT

```
$ openssl x509 -in  github.cer -noout -text
```

在线查看支持 OCSP 封套的网站
```
$ openssl s_client -connect www.example.com:443 -status -tlsextdebug < /dev/null 2>&1
```