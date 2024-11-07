测试环境

```bash
root@doraemon-virtual-machine:~# hostnamectl
 Static hostname: doraemon-virtual-machine
       Icon name: computer-vm
         Chassis: vm
      Machine ID: aaf2113265314ee183ded36068d6dc54
         Boot ID: 4dea440b01e14638ac3c46d648cc2038
  Virtualization: vmware
Operating System: Ubuntu 22.04.5 LTS              
          Kernel: Linux 6.8.0-48-generic
    Architecture: x86-64
 Hardware Vendor: VMware, Inc.
  Hardware Model: VMware Virtual Platform
```



### 网络篇

在网络虚拟环境中，和物理网络中的交换机一样，也需要这样一个软件实现的设备。它需要有很多个虚拟端口，能够将更多的虚拟网卡连接在一起，通过自己的转发功能让这些虚拟网卡之间可以通信。在 Linux 环境下这个软件实现交换机的技术就叫做 bridge （纯软件实现的）

#### 如何使用 bridge

在了解原理之前，研究一下 bridge 的使用还是很有必要的，在 Linux 环境下创建一个小型的虚拟网络，并让它们之间互相通信

##### 创建两个不同的网络

bridge 是用来连接两个不同的虚拟网络，所以需要先使用 net namespace 构建出两个不同的网络空间

![image-20241106093414739](./assets/image-20241106093414739.png)

```bash
# 创建网络空间 net1
ip netns add net1

# 创建一对儿 veth 出来，且将其中的一头 veth1 放入到 net1 中
ip link add veth1 type veth peer name veth1_p
ip link set veth1 netns net1

# 为 veth1 配置 ip，且启动
ip netns exec net1 ip addr add 192.168.0.101/24 dev veth1
ip netns exec net1 ip link set veth1 up

# 查看配置是否成功
ip netns exec net1 ip link list
ip netns exec net1 ifconfig

# 同理，配置 net2
ip netns add net2

ip link add veth2 type veth peer name veth2_p
ip link set veth2 netns net2

ip netns exec net2 ip addr add 192.168.0.102/24 dev veth2
ip netns exec net2 ip link set veth2 up

ip netns exec net2 ip link list
ip netns exec net2 ifconfig
```

```bash
root@doraemon-virtual-machine:~# ip netns exec net1 ip link list
# 1：接口编号，通常是从1开始，依次递增
# lo：接口的名称，代表回环接口
# mtu 65536：最大传输单元，对于回环接口，mtu较大
# qdisc noop：qdisc（队列规则，Queueing Discipline）是指数据包排队的方式，这里使用的是 noop，表示没有特殊的排队策略
# state DOWN：接口的当前状态，处于关闭状态
# mode DEFAULT：接口模式
# group default：网络接口的组
# qlen 1000：接口的队列长度，表示在接口队列中可以排队的最大数据包数
# link/loopback 00:00:00:00:00:00：回环地址，mac地址
# brd 00:00:00:00:00:00：广播地址，回环接口的广播地址通常也是全零
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
# veth1@if3：通过接口 @if3 连接到另一个网络命名空间。通常这种接口用于连接两个网络命名空间或桥接网络
# <NO-CARRIER,BROADCAST,MULTICAST,UP>：接口当前没有连接到物理网络设备，表示没有信号，支持广播，支持多播，接口当前已启用
# state LOWERLAYERDOWN: 状态为 "LOWERLAYERDOWN"，意味着物理连接层（如虚拟网卡和物理网卡）没有连接或者未启用。与之对应的状态是 "LOWER_UP"，表示网络的物理连接已就绪
# link-netnsid 0: 表示该接口属于网络命名空间的 ID，0 表示它属于当前命名空间
4: veth1@if3: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state LOWERLAYERDOWN mode DEFAULT group default qlen 1000
    link/ether 46:6a:96:b5:01:cf brd ff:ff:ff:ff:ff:ff link-netnsid 0
```

##### 把两个网络连接到一起

在创建出两个独立的网络后，这两个网络还不能互相通信，需要创建一个虚拟交换机 bridge，来把这两个网络环境连接起来

![image-20241106101211054](./assets/image-20241106101211054.png)

```bash
# 创建一个 bridge 设置，将刚刚创建的两对 veth 剩下的两头插入到 bridge上
brctl addbr br0
brctl addif br0 veth1_p
brctl addif br0 veth2_p

# 为 br0 分配ip，但分配与否不影响两个网络的互联
# ip addr add 192.168.0.100/24 dev br0

# 将 bridge 启动，且插在其上的 veth 启动起来
ip link set veth1_p up
ip link set veth2_p up
ip link set br0 up

# 网络连通测试
ip netns exec net1 ping 192.168.0.102 -I veth1
```

##### 清理

```bash
ip link delete br0
ip link delete veth1_p
ip link delete veth2_p
ip link list
ip netns del net1
ip netns del net2
ip netns list
```



```bash
# 查看当前的路由表
root@doraemon-virtual-machine:~# ip route
# default：默认路由。意味着所有没有明确匹配其他路由的流量会被转发到这个路由
# via 192.168.137.2: 指定了流量转发的默认网关地址是 192.168.137.2
# dev ens33: 流量将通过 ens33 接口转发，ens33 是物理网络接口
# proto dhcp: 这个路由条目是通过 DHCP 协议动态分配的（通常是通过网络中的 DHCP 服务器自动配置的）
# metric 100: 路由的度量值（metric）。度量值越小，优先级越高，意味着低度量值的路由将优先被选用
default via 192.168.137.2 dev ens33 proto dhcp metric 100

# 169.254.0.0/16: 这是一个特殊的地址范围，称为 链路本地地址（Link-local Address）。当设备无法从 DHCP 服务器获取 IP 地址时，它会自动分配一个 169.254.x.x 地址来进行局部通信
# dev ens33: 这个地址段适用于通过 ens33 接口的流量
# scope link: 该路由是链路本地路由，意味着它仅适用于同一网络段中的设备之间的通信
169.254.0.0/16 dev ens33 scope link metric 1000

# proto kernel: 这个路由是由内核自动生成的，通常是在系统启动时自动配置的
# src 192.168.0.100: 表示通过该路由发送的数据包将使用源 IP 地址 192.168.0.100。这通常是 br0 接口的 IP 地址
192.168.0.0/24 dev br0 proto kernel scope link src 192.168.0.100

# src 192.168.137.128: 表示通过此路由发送的数据包将使用源 IP 地址 192.168.137.128，即 ens33 接口的 IP 地址
192.168.137.0/24 dev ens33 proto kernel scope link src 192.168.137.128 metric 100
```

#### brctl 原理

```c
// bridge-utils-1.7.1/libbridge/libbridge_if.c
int br_add_bridge(const char *brname)
{
    ...
	ret = ioctl(br_socket_fd, SIOCBRADDBR, brname);
    ...
}

int br_del_bridge(const char *brname)
{
	...
	ret = ioctl(br_socket_fd, SIOCBRDELBR, brname);
    ...
}

int br_add_interface(const char *bridge, const char *dev)
{
    ...
	err = ioctl(br_socket_fd, SIOCBRADDIF, &ifr);
    ...
}

int br_del_interface(const char *bridge, const char *dev)
{
    ...
	err = ioctl(br_socket_fd, SIOCBRDELIF, &ifr);
    ...
}
```

综上，bridge-utils 采用 ioctl 函数来向内核发出控制命令，对桥进行各种操作，主要的request如下

```c
// /include/uapi/linux/sockios.h  v5.10.228
/* bridge calls */
#define SIOCBRADDBR     0x89a0		/* create new bridge device     */
#define SIOCBRDELBR     0x89a1		/* remove bridge device         */
#define SIOCBRADDIF	0x89a2		/* add interface to bridge      */
#define SIOCBRDELIF	0x89a3		/* remove interface from bridge */
```

然后研究 ioctl 调用的实际函数，如下

```c
// /net/bridge/br_private.h

int br_add_bridge(struct net *net, const char *name);
int br_del_bridge(struct net *net, const char *name);
int br_add_if(struct net_bridge *br, struct net_device *dev,
	      struct netlink_ext_ack *extack);
int br_del_if(struct net_bridge *br, struct net_device *dev);
```



##### 创建

内核中创建 bridge 的关键代码是 br_add_bridge，位于 /net/bridge/br_if.c 中
