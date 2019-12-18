本书勘误，欢迎大家提交！

页码 | 错误点 | 更正
------------ | ------------- | ---------
P216，6.7.5章节 | ` openssl req -in myreq.pem -noout -verify -key example_csr.pem `|  `openssl req -in example_csr.pem -noout -verify `
P232，6.7.10 章节| 使用不支持 OCSP 封套的 HTTPS 网站进行演示（letsencrypt.org），使用支持 OCSP 封套的 HTTPS 网站进行演示（www.baidu.com） | 写反了，letsencrypt.org 支持 OCSP 封套，而 www.baidu.com 不支持
P148，5.2.1章节 | 第 9 行： `example_cert.pem` 表示 CSR 文件 | 其中的 `example_cert.pem` 应改为 `example_csr.pem`
P158，5.6.2章节 | 添加 HSTS 配置，语句最后少了一个 ";"  | 在语句末尾添加一个分号";"
P107，3.2章节 |  第 9 行： 将主密钥转换为多个密码“快”  | “快”应改为"块"
P107，3.2.1章节 |  倒数第 2 行： 客户端和服务器端持有一个密钥对，在客户端和服务器端连接的时候分别发送给对方 | 改为“客户端和服务器端各持有一份密钥对（公钥+私钥），在它们连接的时候都将自己的公钥发送给对方”似乎更好理解

