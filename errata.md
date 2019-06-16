本书勘误，欢迎大家提交！

页码 | 错误点 | 更正
------------ | ------------- | ---------
P216，6.7.5章节 | ` openssl req -in myreq.pem -noout -verify -key example_csr.pem `|  `openssl req -in example_csr.pem -noout -verify `
P232，6.7.10 章节| 使用不支持 OCSP 封套的 HTTPS 网站进行演示（letsencrypt.org），使用支持 OCSP 封套的 HTTPS 网站进行演示（www.baidu.com） | 写反了，letsencrypt.org 支持 OCSP 封套，而 www.baidu.com 不支持  
