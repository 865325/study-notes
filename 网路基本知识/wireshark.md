### 过滤报文指令

#### IP地址过滤

-   源地址过滤：`ip.src == 192.168.0.1`
-   目的地址过滤：`ip.dst == 192.168.0.1`
-   源或目的地址过滤：`ip.addr == 192.168.0.1` 或 `ip.src == 192.168.0.1 or ip.dst == 192.168.0.1`
-   排除特定 IP 地址：`!(ip.addr == 192.168.0.1)` 或 `ip.addr != 192.168.0.1`
-   IPv6过滤：ipv6.addr == ff02::fb

#### 端口过滤

-   特定端口：`tcp.port == 80`
-   端口范围：`tcp.port >= 2048`
-   源端口或目的端口：`tcp.srcport == 12345 && tcp.dstport == 80` 

#### MAC过滤

-   特定MAC地址：`eth.dst == A0:00:00:04:C5:84` 或 `eth.src==A0-00-00-04-C5-84`

#### 协议过滤

-   捕获特定协议的数据包： `tcp` `udp` `arp` `icmp` `http` `smtp` `ftp` `dns` `msnms` `ip`  `oicq` `dhcp`
-   排除特定协议：`not arp` 或 `!tcp` 

#### 长度和内容过滤

-   数据段长度：`udp.length < 30`
-   内容匹配：`http.request.uri matches "vipscu"` 

#### HTTP 特定请求过滤

-   GET 请求：`http.request.method == GET`
-   POST 请求：`http.request.method == POST` 
-   响应包：`http contains "Content-Type:"`

#### 类型过滤

-   Ethernet II：`eth.type == 0x0800`