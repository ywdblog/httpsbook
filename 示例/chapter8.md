1：8.9.1 小节

tcpdump 捕获流量：

```
$ tcpdump -s 0  -i eth1  port 443 and host 10.235.173.30 -w https.pcap  
```

2：8.9.2 小节

（1）显示握手失败的 TLS/SSL 消息（图8-16）

[wireshark 8.9.2-1.pcap 包](res/8.9.2-1.pcap)

（2）RSA 密钥交换的例子

[wireshark 8.9.2-2.pcap 包](res/8.9.2-2.pcap)

（3）ECDHE 密钥交换的例子

[wireshark 8.9.2-3.pcap 包](res/8.9.2-3.pcap)

（4）基于 Session ID 会话恢复的例子

[wireshark 8.9.2-4.pcap 包](res/8.9.2-4.pcap)

（5）基于 Session Ticket 会话恢复的例子

[wireshark 8.9.2-5.pcap 包](res/8.9.2-5.pcap)