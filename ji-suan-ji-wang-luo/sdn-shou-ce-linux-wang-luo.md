# SDN手册-Linux网络

## Linux网络配置

### 配置IP地址

```bash
# 使用ifconfig
ifconfig eth0 192.168.1.3 netmask 255.255.255.0

# 使用用ip命令增加一个IP
ip addr add 192.168.1.4/24 dev eth0

# 使用ifconfig增加网卡别名
ifconfig eth0:0 192.168.1.10
```

这样配置的`IP`地址重启机器后会丢失，所以一般应该把网络配置写入文件中。如`Ubuntu`可以将网卡配置写入`/etc/network/interfaces`（`Redhat`和`CentOS`则需要写入`/etc/sysconfig/network-scripts/ifcfg-eth0`中）：

```bash
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
    address 192.168.1.3
    netmask 255.255.255.0
    gateway 192.168.1.1 #网关

auto eth1
iface eth1 inet dhcp
```

```
1. "auto lo"和"iface lo inet loopback"表示启用并配置本地回环接口（loopback）。
2. "auto eth0"表示启用eth0网络接口。
3. "iface eth0 inet static"表示将eth0配置为静态IP地址。
4. "address 192.168.1.3"指定eth0的IP地址为192.168.1.3。
5. "netmask 255.255.255.0"指定eth0的子网掩码为255.255.255.0。
6. "gateway 192.168.1.1"指定eth0的网关为192.168.1.1。

7. "auto eth1"表示启用eth1网络接口。
8. "iface eth1 inet dhcp"表示将eth1配置为使用DHCP获取IP地址。
```

[什么是网关，网关的作用是什么？ - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/165142303)

```
通常情况下，一个网络接口的网关设置为本地网络中的路由器或防火墙的IP地址。这个IP地址通常是在同一子网内，并且是网络管理员根据网络拓扑和需求进行配置的。

常见的网关设置是将最后一位IP地址设置为1，例如192.168.1.1。但这并不是绝对的规定，具体的网关设置取决于网络的配置和需求。

在一些情况下，可能会有多个网关设置，例如在使用多个路由器进行负载均衡或实现冗余的情况下。这种情况下，每个网关的IP地址可能会有所不同。

因此，一个网络接口的网关通常根据具体网络配置和需求而定，可以是本地网络中的路由器或防火墙的IP地址，并且可能会有多个网关设置。
```

### 配置默认路由

```sh
# 使用route命令
route add default gw 192.168.1.1
# 也可以使用ip命令
ip route add default via 192.168.1.1
```

配置默认路由和配置网关是一回事吗？

```bash
配置默认路由和配置网关在某种程度上是相同的，因为它们都涉及将数据包从本地网络发送到其他网络。

默认路由是指在路由表中设置的一个特殊的路由规则，用于指示当目标地址不在本地子网内时，数据包应该被发送到哪个网关。默认路由通常被称为0.0.0.0/0，表示所有目标地址。

配置网关是指将一个特定的IP地址指定为本地网络中的网关设备。这个网关设备负责将数据包从本地网络传递到其他网络。

在大多数情况下，配置默认路由和配置网关是相同的操作。当设置默认路由时，通常会指定一个网关的IP地址，以便将数据包发送到其他网络。这个网关的IP地址就是配置的网关。

因此，虽然在概念上有些微妙的区别，但在实际操作中，配置默认路由和配置网关可以视为相同的操作，都是为了实现将数据包从本地网络发送到其他网络。
```

### 配置VLAN

```sh
# 安装并加载内核模块
apt-get install vlan
modprobe 8021q

# 添加vlan
vconfig add eth0 100
ifconfig eth0.100 192.168.100.2 netmask 255.255.255.0

# 删除vlan
vconfig rem eth0.100
```

```bash
1. "apt-get install vlan" 是一个Debian或Ubuntu系统中的命令，用于通过包管理器apt-get安装vlan软件包。这个命令会从软件源中下载并安装vlan工具，以便在系统中配置和管理VLAN。

2. "modprobe 8021q" 是一个Linux内核模块加载命令，用于加载8021q内核模块。8021q模块是用于支持VLAN的内核模块，通过加载它，系统可以使用VLAN功能。
3. "vconfig add eth0 100" 是一个命令，用于在eth0网络接口上创建一个VLAN接口，VLAN标识符为100。这将创建一个名为eth0.100的虚拟接口，用于处理VLAN ID为100的网络流量。

4. "ifconfig eth0.100 192.168.100.2 netmask 255.255.255.0" 是一个命令，用于为刚刚创建的eth0.100 VLAN接口配置IP地址和子网掩码。这将为eth0.100接口分配IP地址192.168.100.2，子网掩码为255.255.255.0。
```

VLAN的ip和掩码的设置跟网络接口上的ip和掩码有什么关联？

```bash
VLAN的IP地址和子网掩码是针对虚拟局域网（VLAN）接口的配置，而网络接口的IP地址和子网掩码是针对物理网络接口的配置。

当你在物理网络接口上创建一个VLAN接口时，该VLAN接口将成为物理接口的虚拟子接口。这个虚拟子接口将具有自己的IP地址和子网掩码，用于在该VLAN内进行通信。

VLAN的IP地址和子网掩码与物理接口上的IP地址和子网掩码是相互独立的。它们在逻辑上是分开的，因为VLAN是一种虚拟隔离的网络。每个VLAN都可以有自己的IP地址和子网掩码，与其他VLAN和物理网络接口上的地址无关。

通过为VLAN接口配置独立的IP地址和子网掩码，可以实现不同VLAN之间的互联和通信。这样，每个VLAN都可以拥有自己的IP地址空间，进行独立的网络通信。

总结起来，VLAN的IP地址和子网掩码与物理网络接口上的IP地址和子网掩码是独立的，用于在VLAN内部进行通信和隔离。每个VLAN都可以有自己的网络配置，与其他VLAN和物理接口无关。
```

### 配置硬件选项

```sh
# 改变speed
ethtool -s eth0 speed 1000 duplex full

# 关闭GRO
ethtool -K eth0 gro off

# 开启网卡多队列
ethtool -L eth0 combined 4

# 开启vxlan offload
ethtool -K ens2f0 rx-checksum on
ethtool -K ens2f0 tx-udp_tnl-segmentation on

# 查询网卡统计
ethtool -S eth0
```

```bash
1. "ethtool -s eth0 speed 1000 duplex full" 是一个命令，用于将eth0网络接口的速度设置为1000Mbps（千兆位每秒），并将双工模式设置为全双工。这将改变网络接口的传输速率和双工模式。

2. "ethtool -K eth0 gro off" 是一个命令，用于关闭eth0网络接口的GRO（Generic Receive Offload）功能。GRO是一种在网络接收端进行数据包重组的技术，通过关闭它可以改变网络接口的处理方式。

3. "ethtool -L eth0 combined 4" 是一个命令，用于开启eth0网络接口的多队列功能，并将队列数设置为4。多队列可以提高网络接口的并发处理能力。

4. "ethtool -K ens2f0 rx-checksum on" 和 "ethtool -K ens2f0 tx-udp_tnl-segmentation on" 是两个命令，用于开启ens2f0网络接口的VXLAN（Virtual Extensible LAN）卸载功能。这些选项可以提高网络接口对VXLAN封装和解封装的性能。

5. "ethtool -S eth0" 是一个命令，用于查询eth0网络接口的统计信息。它将显示一些与网络接口相关的统计数据，如接收和发送的数据包数量、错误计数等。
```

## 虚拟网络设备

Linux提供了许多虚拟设备，这些虚拟设备有助于构建复杂的网络拓扑，满足各种网络需求。

### 网桥

网桥是一个二层设备，工作在链路层，主要是根据MAC学习来转发数据到不同的port。

```bash
现代的网桥通常是以交换机的形式存在，具有更高级的功能和性能。以下是一些现代网桥的特点：

1. 多端口：现代网桥通常有多个端口，可以连接多个设备，以实现更大规模的网络连接。

2. 自学习：现代网桥可以通过自学习算法学习网络中各个设备的MAC地址，并将其存储在一个MAC地址表中。这样，网桥可以根据目的MAC地址来决定数据包的转发路径，提高数据传输的效率。

3. 转发和过滤：网桥可以根据MAC地址表来决定数据包的转发路径，只将数据包转发到目标设备所在的端口，从而减少网络中不必要的数据传输。

4. VLAN支持：现代网桥支持虚拟局域网（VLAN）的功能，可以将一个物理局域网划分为多个逻辑上的虚拟局域网，从而提高网络的安全性和性能。

5. QoS支持：网桥可以支持服务质量（QoS）功能，通过对数据包进行优先级和带宽的管理，提供更好的网络性能和用户体验。

6. 网络管理：现代网桥通常提供网络管理接口，可以通过网络管理软件对网桥进行配置、监控和故障排除。

总之，现代网桥具有更高级的功能和性能，可以更好地管理和优化网络传输。它们在企业和家庭网络中得到广泛应用。
```

```bash
Linux的网桥是一种虚拟网络设备。在Linux系统中，可以使用bridge工具创建和管理网桥设备。

网桥设备是一种虚拟设备，它可以连接多个物理或虚拟网络接口，并将它们组合成一个逻辑上的网络。网桥通过学习和转发MAC地址，实现不同网络接口之间的数据传输。它可以让连接到不同网桥接口的设备之间通信，就像它们连接在同一个网络中一样。

网桥设备在Linux系统中使用广泛，特别是在构建虚拟化环境或实现网络分段和隔离时。通过配置网桥，可以实现虚拟机之间的通信、虚拟机与物理网络之间的通信等。

在Linux中，可以使用命令行工具如brctl或ip命令来创建、配置和管理网桥设备。此外，还可以通过配置网络接口配置文件（如/etc/network/interfaces）或网络管理工具（如NetworkManager）来设置网桥。
```

```sh
# 创建网桥
brctl addbr br0
# 添加设备到网桥
brctl addif br0 eth1
# 查询网桥mac表
brctl showmacs br0
```

### veth

veth pair是一对虚拟网络设备，一端发送的数据会由另外一端接受，常用于不同的网络命名空间。

```sh
# 创建veth pair
ip link add veth0 type veth peer name veth1

# 将veth1放入另一个netns
ip link set veth1 netns newns

这段代码演示了如何创建一个veth pair，并将其中一个端口（veth1）放入另一个网络命名空间（newns）中。

首先，使用`ip link add`命令创建一个veth pair，命名为veth0和veth1。这将创建两个虚拟网络接口，它们通过一个虚拟以太网链路连接在一起。

然后，使用`ip link set`命令将veth1端口移动到指定的网络命名空间（newns）。这将使veth1端口从当前网络命名空间中移动到新的网络命名空间中，使其在新的网络命名空间中可见和可用。

通过这样的操作，你可以在不同的网络命名空间中创建和管理虚拟网络接口，实现网络隔离和通信。在这个例子中，veth0端口仍然留在当前网络命名空间中，而veth1端口已经移动到新的网络命名空间（newns）中。
```

```bash
在Linux中，veth（Virtual Ethernet）是一种虚拟网络设备对，用于创建一对虚拟网络接口。veth设备通常用于连接两个网络命名空间（network namespace）之间，以实现虚拟化环境中的网络通信。

veth设备由两个端口组成，一个端口位于一个网络命名空间，另一个端口位于另一个网络命名空间。这两个端口通过一个虚拟以太网链路连接在一起，就像一根虚拟的以太网电缆。

veth设备可以用于各种用途，例如：

1. 容器网络：当创建一个容器时，可以使用veth设备将容器内部的网络接口连接到宿主机的网络接口，从而实现容器与宿主机以及其他容器之间的通信。

2. 虚拟机网络：在虚拟化环境中，可以使用veth设备将虚拟机的网络接口连接到宿主机的网络接口，实现虚拟机与宿主机以及其他虚拟机之间的通信。

3. 网络隔离：通过将不同网络命名空间中的veth设备连接到Linux的网桥或路由器上，可以实现网络隔离和分段，确保不同网络之间的通信只能通过指定的网桥或路由器进行。

在Linux中，可以使用ip命令或网络管理工具（如Docker、Kubernetes等）来创建和管理veth设备。
```

```bash
veth和网桥是两个不同的概念，它们在网络中扮演不同的角色。

1. veth（Virtual Ethernet）：veth是一对虚拟网络接口设备，用于连接两个网络命名空间（network namespace）。veth设备通常用于虚拟化环境中，如容器化和虚拟机环境中，用于连接容器或虚拟机的网络接口到宿主机的网络接口。veth设备通过一个虚拟以太网链路将两个网络接口连接在一起，实现数据的传输和通信。

2. 网桥（Bridge）：网桥是一种网络设备，用于连接两个或多个局域网（LAN），以便它们可以相互通信。网桥通过学习和转发MAC地址，将数据包从一个端口转发到另一个端口，实现不同网络接口之间的数据传输。网桥可以连接物理网络接口和虚拟网络接口，如veth设备、虚拟机接口等。

区别如下：

1. 功能：veth设备用于连接不同网络命名空间中的网络接口，实现虚拟化环境中的网络通信。而网桥用于连接不同的局域网，实现不同网络接口之间的数据传输。

2. 范围：veth设备通常用于虚拟化环境，如容器化和虚拟机环境。而网桥可以用于各种网络环境，包括物理网络和虚拟网络。

3. 数据传输方式：veth设备通过虚拟以太网链路将两个网络接口连接在一起，实现数据的传输。而网桥通过学习和转发MAC地址，将数据包从一个端口转发到另一个端口，实现数据的传输。

总之，veth和网桥在功能、范围和数据传输方式上有所不同，但它们都是在网络中扮演重要角色的设备。
```

```bash
不同网络命名空间（network namespace）和局域网（LAN）是两个不同的概念，它们在网络中的范围和作用上有所不同。

1. 不同网络命名空间：网络命名空间是Linux内核提供的一种机制，用于将网络资源隔离开来，使其在不同的命名空间中运行的进程可以有独立的网络栈和网络接口。每个网络命名空间都有自己的网络接口、路由表、ARP表和网络配置。不同网络命名空间之间的网络资源是隔离的，它们可以有不同的IP地址、MAC地址和网络配置。通过不同网络命名空间，可以实现网络隔离、虚拟化环境和容器化等功能。

2. 局域网：局域网是指在一个较小的地理范围内，由多个计算机和网络设备组成的网络。局域网中的设备可以通过物理介质（如以太网）或无线连接在一起，并共享资源和进行通信。局域网通常由一个或多个交换机或网桥连接起来，以实现设备之间的数据传输和通信。局域网内的设备通常使用相同的IP地址段和子网掩码，可以直接通过MAC地址进行通信。

不同网络命名空间和局域网的区别如下：

1. 范围：不同网络命名空间是在操作系统内部创建的，用于隔离和管理不同网络资源。而局域网是一个实际的物理或逻辑网络，由多个设备和网络设备组成。

2. 隔离性：不同网络命名空间之间的网络资源是隔离的，它们可以有不同的IP地址、MAC地址和网络配置。而局域网内的设备通常使用相同的IP地址段和子网掩码，可以直接进行通信。

3. 功能：不同网络命名空间可以用于实现网络隔离、虚拟化环境和容器化等功能。而局域网主要用于在一个较小的地理范围内，使设备可以共享资源和进行通信。

总之，不同网络命名空间和局域网在范围、隔离性和功能上有所不同，但它们都是在网络中扮演重要角色的概念。
```

### TAP/TUN

TAP/TUN设备是一种让用户态程序向内核协议栈注入数据的设备，TAP等同于一个以太网设备，工作在二层；而TUN则是一个虚拟点对点设备，工作在三层。

```sh
ip tuntap add tap0 mode tap
ip tuntap add tun0 mode tun

这两个命令是在Linux系统下使用ip命令创建TAP和TUN设备。
第一个命令`ip tuntap add tap0 mode tap`创建了一个名为tap0的TAP设备，工作模式为tap。这将在系统中创建一个虚拟的以太网接口，可用于建立虚拟局域网或实现网络桥接。
第二个命令`ip tuntap add tun0 mode tun`创建了一个名为tun0的TUN设备，工作模式为tun。这将在系统中创建一个虚拟的点对点网络接口，可用于实现虚拟私有网络（VPN）或其他网络隧道技术。
这些命令创建的设备可以在系统中进行配置和使用，例如分配IP地址、设置路由等。可以使用其他命令如ip addr、ip route等来进一步配置这些设备。
```

```bash
TAP（Tap-Win32 Adapter V9）和TUN（Tunneling）是两种虚拟网络设备驱动程序，用于在计算机网络中创建虚拟网络接口。
TAP是一种虚拟以太网适配器，它可以模拟一个以太网接口，使得计算机可以像使用物理以太网适配器一样使用它。TAP适配器通常用于实现虚拟专用网络（VPN）和其他网络隧道技术，可以在计算机之间创建安全的通信通道。
TUN是一种虚拟点对点网络接口，它可以模拟一个点对点的连接，允许两个计算机之间直接通信。TUN适配器通常用于实现虚拟私有网络（VPN）和其他网络隧道技术，可以在计算机之间创建安全的通信通道。
TAP和TUN适配器可以通过软件实现，也可以通过硬件设备实现。它们在网络安全、虚拟化和网络隧道技术等领域中被广泛使用。

当我们在计算机网络中进行通信时，数据包通常通过物理网络适配器（如以太网适配器）发送和接收。但是，有时我们需要创建虚拟网络接口，以便在计算机之间建立安全的通信通道。
这就是TAP（Tap-Win32 Adapter V9）和TUN（Tunneling）的作用。它们是虚拟网络设备驱动程序，可以模拟一个虚拟网络接口，使得计算机可以像使用物理网络适配器一样使用它们。
TAP适配器是一种虚拟以太网适配器。当我们需要创建一个虚拟专用网络（VPN）或其他网络隧道时，我们可以使用TAP适配器。它可以模拟一个以太网接口，允许计算机将数据包发送到虚拟网络中的其他计算机，就像它们连接在同一个物理以太网上一样。
TUN适配器是一种虚拟点对点网络接口。当我们需要创建一个虚拟私有网络（VPN）或其他网络隧道时，我们可以使用TUN适配器。它可以模拟一个点对点的连接，允许两个计算机之间直接通信，就像它们之间有一条专用的物理连接一样。
通过使用TAP和TUN适配器，我们可以在计算机之间创建安全的通信通道，保护数据的传输和隐私。这对于构建安全的VPN连接、虚拟化网络环境以及实现网络隧道技术非常有用。

TUN和TAP的区别在于它们模拟的网络层级和工作方式：
1. 网络层级：TUN适配器工作在网络层（第三层），而TAP适配器工作在数据链路层（第二层）。这意味着TUN适配器可以处理IP数据包，而TAP适配器可以处理以太网帧。
2. 工作方式：TUN适配器模拟了一个点对点的连接，它接收来自操作系统的IP数据包，并将其封装为点对点隧道。这使得两台计算机之间可以直接通信，就像它们之间有一条专用的物理连接一样。TAP适配器模拟了一个以太网接口，它接收来自操作系统的以太网帧，并将其发送到虚拟网络中的其他计算机。这使得多台计算机可以连接到同一个虚拟网络，就像它们连接在同一个物理以太网上一样。
3. 用途：由于TUN适配器工作在网络层，它通常用于实现虚拟私有网络（VPN）和其他网络隧道技术，用于加密和隔离网络流量。TAP适配器工作在数据链路层，它通常用于创建虚拟局域网（VLAN）或虚拟以太网，用于建立虚拟网络环境和实现网络桥接。
总结起来，TUN适配器适用于点对点的通信，处理IP数据包，常用于VPN和网络隧道；而TAP适配器适用于多台计算机连接到同一个虚拟网络，处理以太网帧，常用于虚拟局域网和网络桥接。
```

## iptables/netfilter

`iptables`是一个配置`Linux`内核防火墙的命令行工具，它基于内核的`netfilter`机制。新版本的内核（3.13+）也提供了`nftables`，用于取代`iptables`。

### netfilter

`netfilter`是`Linux`内核的包过滤框架，它提供了一系列的钩子（`Hook`）供其他模块控制包的流动。这些钩子包括

* `NF_IP_PRE_ROUTING`：刚刚通过数据链路层解包进入网络层的数据包通过此钩子，它在路由之前处理
* `NF_IP_LOCAL_IN`：经过路由查找后，送往本机（目的地址在本地）的包会通过此钩子
* `NF_IP_FORWARD`：不是本地产生的并且目的地不是本地的包（即转发的包）会通过此钩子
* `NF_IP_LOCAL_OUT`：所有本地生成的发往其他机器的包会通过该钩子
* `NF_IP_POST_ROUTING`：在包就要离开本机之前会通过该钩子，它在路由之后处理

![netfilter](https://tonydeng.gitbooks.io/sdn/content/linux/images/netfilter.png)

```
Netfilter是一个在Linux操作系统中实现的网络数据包过滤框架。它允许系统管理员通过配置规则来控制网络数据包的流动。

Netfilter的主要功能包括：
1. 数据包过滤：Netfilter可以根据预定义的规则，对进出系统的网络数据包进行过滤和筛选。
2. 网络地址转换（NAT）：Netfilter可以实现网络地址转换，将内部网络的私有IP地址映射为公共IP地址，实现内网与外网的通信。
3. 数据包修改：Netfilter可以修改数据包的源地址、目的地址、端口等信息，实现网络数据包的伪装和重定向。
4. 连接跟踪：Netfilter可以跟踪网络连接的状态，对于已建立的连接，可以根据连接的状态进行过滤和处理。

使用Netfilter，可以通过iptables工具来配置和管理规则。iptables是一个命令行工具，用于在Linux系统中设置和管理防火墙规则。

以下是一些常用的iptables命令：
- iptables -A INPUT -p tcp --dport 22 -j ACCEPT：允许通过SSH协议的连接。
- iptables -A INPUT -p tcp --dport 80 -j ACCEPT：允许通过HTTP协议的连接。
- iptables -A INPUT -j DROP：拒绝所有其他输入连接。

需要注意的是，使用Netfilter和iptables需要具备一定的网络和系统知识，不正确的配置可能会导致网络不可用或安全漏洞。建议在使用之前先了解相关文档或咨询专业人士。
```

### iptables

`iptables`通过表和链来组织数据包的过滤规则，每条规则都包括匹配和动作两部分。默认情况下，每张表包括一些默认链，用户也可以添加自定义的链，这些链都是顺序排列的。这些表和链包括：

* `raw表`用于决定数据包是否被状态跟踪机制处理，内建`PREROUTING`和`OUTPUT`两个链
* `filter表`用于过滤，内建`INPUT`（目的地是本地的包）、`FORWARD`（不是本地产生的并且目的地不是本地）和`OUTPUT`（本地生成的包）等三个链
* `nat表`用于网络地址转换，内建`PREROUTING`（在包刚刚到达防火墙时改变它的目的地址）、`INPUT`、`OUTPUT`和`POSTROUTING`（要离开防火墙之前改变其源地址）等链
* `mangle表`用于对报文进行修改，内建`PREROUTING`、`INPUT`、`FORWARD`、`OUTPUT`和`POSTROUTING`等链
* `security表`用于根据安全策略处理数据包，内建`INPUT`、`FORWARD`和`OUTPUT`链

| Tables↓/Chains→               | PREROUTING | INPUT | FORWARD | OUTPUT | POSTROUTING |
| ----------------------------- | :--------: | :---: | :-----: | :----: | :---------: |
| (routing decision)            |            |       |         |    ✓   |             |
| **raw**                       |      ✓     |       |         |    ✓   |             |
| (connection tracking enabled) |      ✓     |       |         |    ✓   |             |
| **mangle**                    |      ✓     |   ✓   |    ✓    |    ✓   |      ✓      |
| **nat** (DNAT)                |      ✓     |       |         |    ✓   |             |
| (routing decision)            |      ✓     |       |         |    ✓   |             |
| **filter**                    |            |   ✓   |    ✓    |    ✓   |             |
| **security**                  |            |   ✓   |    ✓    |    ✓   |             |
| **nat** (SNAT)                |            |   ✓   |         |        |      ✓      |

所有链默认都是没有任何规则的，用户可以按需要添加规则。每条规则都包括匹配和动作两部分：

* 匹配可以有多条，比如匹配端口、`IP`、数据包类型等。匹配还可以包括模块（如`conntrack`、`recent`等），实现更复杂的过滤。
* 动作只能有一个，通过`-j`指定，如`ACCEPT`、`DROP`、`RETURN`、`SNAT`、`DNAT`等

这样，网络数据包通过iptables的过程为:

![iptables](https://tonydeng.gitbooks.io/sdn/content/linux/images/iptables.png)

其规律为

1. 当一个数据包进入网卡时，数据包首先进入`PREROUTING`链，在`PREROUTING`链中我们有机会修改数据包的`DestIP`(目的IP)，然后内核的"路由模块"根据"数据包目的IP"以及"内核中的路由表" 判断是否需要转送出去(注意，这个时候数据包的`DestIP`有可能已经被我们修改过了)
2. 如果数据包就是进入本机的(即数据包的目的IP是本机的网口IP)，数据包就会沿着图向下移动，到达`INPUT`链。数据包到达`INPUT`链后，任何进程都会收到它
3. 本机上运行的程序也可以发送数据包，这些数据包经过`OUTPUT`链，然后到达`POSTROTING`链输出(注意，这个时候数据包的`SrcIP`有可能已经被我们修改过了)
4. 如果数据包是要转发出去的(即目的IP地址不再当前子网中)，且内核允许转发，数据包就会向右移动，经过`FORWARD`链，然后到达`POSTROUTING`链输出(选择对应子网的网口发送出去)

**【iptables示例】**

*   查看规则列表

    ```sh
    iptables -nvL
    ```
*   允许22端口

    ```sh
    iptables -A INPUT -p tcp --dport 22 -j ACCEPT

    这条iptables命令是用于在防火墙规则中允许通过TCP协议的SSH连接（端口号22）。
    具体解释如下：
    - `-A INPUT`：将规则添加到INPUT链，该链用于处理进入系统的数据包。
    - `-p tcp`：指定规则应用于TCP协议的数据包。
    - `--dport 22`：指定目标端口为22，即SSH服务的默认端口。
    - `-j ACCEPT`：如果数据包符合规则，就接受它，允许通过防火墙。
    这条规则的作用是允许通过SSH协议连接到系统的22端口。
    ```
*   允许来自192.168.0.4的包

    ```sh
    iptables -A INPUT -s 192.168.0.4 -j ACCEPT

    这条iptables命令是用于在防火墙规则中允许特定源IP地址（192.168.0.4）的数据包通过。
    具体解释如下：
    - `-A INPUT`：将规则添加到INPUT链，该链用于处理进入系统的数据包。
    - `-s 192.168.0.4`：指定源IP地址为192.168.0.4，即只允许来自该IP地址的数据包通过。
    - `-j ACCEPT`：如果数据包符合规则，就接受它，允许通过防火墙。
    这条规则的作用是允许来自IP地址为192.168.0.4的数据包通过防火墙。可以根据需要修改源IP地址或其他规则来适应特定的网络配置和安全需求。
    ```
*   允许现有连接或与现有连接关联的包

    ```sh
    iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

    这条iptables命令是用于在防火墙规则中允许与已建立的连接或相关的连接的数据包通过。
    具体解释如下：
    - `-A INPUT`：将规则添加到INPUT链，该链用于处理进入系统的数据包。
    - `-m state --state ESTABLISHED,RELATED`：使用state模块来匹配连接的状态，其中"ESTABLISHED"表示已经建立的连接，"RELATED"表示与已建立连接相关的数据包。
    - `-j ACCEPT`：如果数据包符合规则，就接受它，允许通过防火墙。
    这条规则的作用是允许与已经建立的连接或相关的连接的数据包通过防火墙。这对于维护已建立的连接的通信流畅性非常重要，因为它允许回应和响应已经建立的连接的数据包。
    这个规则通常用于防火墙的入站规则，以确保已建立的连接能够正常通信。
    ```
*   禁止ping包

    ```sh
    iptables -A INPUT -p icmp --icmp-type echo-request -j DROP


    这条iptables命令是用于在防火墙规则中禁止通过ICMP协议的Echo请求（ping）。
    具体解释如下：
    - `-A INPUT`：将规则添加到INPUT链，该链用于处理进入系统的数据包。
    - `-p icmp`：指定规则应用于ICMP协议的数据包。
    - `--icmp-type echo-request`：指定ICMP类型为Echo请求，即ping请求。
    - `-j DROP`：如果数据包符合规则，就丢弃它，防止通过防火墙。
    这条规则的作用是禁止通过防火墙的ICMP Echo请求（ping）。这可以用于限制对系统的ping请求，从而增加系统的安全性和隐私性。
    需要注意的是，禁用ping可能会影响网络诊断和连接测试，因此在实际使用中应谨慎考虑是否禁用ICMP Echo请求。
    ```
*   禁止所有其他包

    ```sh
    iptables -P INPUT DROP
    iptables -P FORWARD DROP
    ```
*   MASQUERADE

    ```sh
    iptables -t nat -I POSTROUTING -s 10.0.0.30/32 -j MASQUERADE

    这条iptables命令是用于在NAT表中添加一个POSTROUTING规则，将源IP地址为10.0.0.30的数据包进行MASQUERADE（伪装）处理。
    具体解释如下：
    - `-t nat`：指定使用NAT表。
    - `-I POSTROUTING`：在POSTROUTING链中插入规则，该链用于处理从本地系统发送到外部网络的数据包。
    - `-s 10.0.0.30/32`：指定源IP地址为10.0.0.30，即只针对该IP地址的数据包进行处理。
    - `-j MASQUERADE`：如果数据包符合规则，就进行MASQUERADE处理，即将源IP地址替换为防火墙外部接口的IP地址，以实现网络地址转换（NAT）。
    这条规则的作用是将源IP地址为10.0.0.30的数据包进行MASQUERADE处理，使其在通过防火墙后具有防火墙外部接口的IP地址作为源IP地址。这通常用于实现网络地址转换，将内部私有IP地址转换为外部公共IP地址，以便内部主机与外部网络进行通信。
    ```
*   NAT

    ```sh
    iptables -I FORWARD -m state --state RELATED,ESTABLISHED -j ACCEPT
    iptables -I INPUT   -m state --state RELATED,ESTABLISHED -j ACCEPT
    iptables -t nat -I OUTPUT -d 55.55.55.55/32 -j DNAT --to-destination 10.0.0.30
    iptables -t nat -I PREROUTING -d 55.55.55.55/32 -j DNAT --to-destination 10.0.0.30
    iptables -t nat -I POSTROUTING -s 10.0.0.30/32 -j SNAT --to-source 55.55.55.55


    这组iptables命令用于设置网络地址转换（NAT）规则，以实现特定IP地址的目标地址转换和源地址转换。
    具体解释如下：
    1. `iptables -I FORWARD -m state --state RELATED,ESTABLISHED -j ACCEPT`：在FORWARD链中插入规则，允许与已建立的连接或相关的连接的数据包通过防火墙。
    2. `iptables -I INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT`：在INPUT链中插入规则，允许与已建立的连接或相关的连接的数据包通过防火墙。
    3. `iptables -t nat -I OUTPUT -d 55.55.55.55/32 -j DNAT --to-destination 10.0.0.30`：在OUTPUT链中插入规则，将目标IP地址为55.55.55.55的数据包进行目标地址转换，将其转发到10.0.0.30。
    4. `iptables -t nat -I PREROUTING -d 55.55.55.55/32 -j DNAT --to-destination 10.0.0.30`：在PREROUTING链中插入规则，将目标IP地址为55.55.55.55的数据包进行目标地址转换，将其转发到10.0.0.30。
    5. `iptables -t nat -I POSTROUTING -s 10.0.0.30/32 -j SNAT --to-source 55.55.55.55`：在POSTROUTING链中插入规则，将源IP地址为10.0.0.30的数据包进行源地址转换，将其转发到55.55.55.55。
    这组规则的作用是允许与已建立的连接或相关的连接的数据包通过防火墙，并设置目标地址转换和源地址转换规则，以实现特定IP地址的转换。具体的转换行为和IP地址可以根据实际需求进行修改。
    ```
*   端口映射

    ```sh
    iptables -t nat -I OUTPUT -d 55.55.55.55/32 -p tcp -m tcp --dport 80 -j DNAT --to-destination 10.10.10.3:80
    iptables -t nat -I POSTROUTING -m conntrack ! --ctstate DNAT -j ACCEPT
    iptables -t nat -I PREROUTING -d 55.55.55.55/32 -p tcp -m tcp --dport 80 -j DNAT --to-destination 10.10.10.3:80


    第一条命令：
    iptables -t nat -I OUTPUT -d 55.55.55.55/32 -p tcp -m tcp --dport 80 -j DNAT --to-destination 10.10.10.3:80
    这条命令将目标地址为55.55.55.55，目标端口为80的TCP数据包重定向到10.10.10.3的80端口。这意味着当有TCP数据包从本机发往55.55.55.55的80端口时，它们将被重定向到10.10.10.3的80端口。
    第二条命令：
    iptables -t nat -I POSTROUTING -m conntrack ! --ctstate DNAT -j ACCEPT
    这条命令允许通过NAT转换的数据包离开防火墙。它使用conntrack模块来检查数据包的连接状态，如果数据包不是由DNAT转换产生的，就允许它通过。
    第三条命令：
    iptables -t nat -I PREROUTING -d 55.55.55.55/32 -p tcp -m tcp --dport 80 -j DNAT --to-destination 10.10.10.3:80
    这条命令将目标地址为55.55.55.55，目标端口为80的TCP数据包重定向到10.10.10.3的80端口。这意味着当有TCP数据包从其他主机发往本机的55.55.55.55的80端口时，它们将被重定向到10.10.10.3的80端口。

    这段代码实现了端口映射。它将目标地址为55.55.55.55的TCP数据包的目标端口80重定向到本地地址10.10.10.3的80端口。这样，当有TCP数据包发送到55.55.55.55的80端口时，它们将被转发到10.10.10.3的80端口上。这种方式可以用于将外部请求转发到内部服务器上，实现网络服务的访问。
    ```
*   重置所有规则

    ```sh
    iptables -F
    iptables -t nat -F
    iptables -t mangle -F
    iptables -X


    这些命令用于清除iptables规则和链。

    - `iptables -F`：清除所有的iptables规则，包括默认规则和自定义规则。
    - `iptables -t nat -F`：清除所有的nat表中的规则，包括默认规则和自定义规则。nat表用于网络地址转换。
    - `iptables -t mangle -F`：清除所有的mangle表中的规则，包括默认规则和自定义规则。mangle表用于修改数据包的特定字段。
    - `iptables -X`：删除所有自定义的链，这些链不再被引用。

    通过执行这些命令，可以将iptables配置恢复到初始状态，以便重新配置新的规则和链。
    ```

### nftables

`nftables` 是从内核 3.13 版本引入的新的数据包过滤框架，旨在替代现用的 `iptables` 框架。`nftables`引入了一个新的命令行工具`nft`，取代了之前的`iptables`、`ip6iptables`、`ebtables`等各种工具。跟`iptables`相比，`nftables`带来了一系列的好处:

* 更易用易理解的语法
* 表和链是完全可配置的
* 匹配和目标之间不再有区别
* 在一个规则中可以定义多个动作
* 每个链和规则都没有内建的计数器
* 更好的动态规则集更新支持
* 简化`IPv4/IPv6`双栈管理
* 支持`set/map`等
* 支持级连（需要内核4.1+）

跟`iptables`类似，`nftables`也是使用表和链来管理规则。其中，表包括`ip`、`arp`、`ip6`、`bridge`、`inet`和`netdev`等6个类型。下面是一些简单的例子

```sh
# 新建一个ip类型的表
nft add table ip foo

# 列出所有表
nft list tables

# 删除表
nft delete table ip foo

# 添加链
nft add table ip filter
nft add chain ip filter input { type filter hook input priority 0 \; }
nft add chain ip filter output { type filter hook output priority 0 \; }

# 添加规则
nft add rule filter output ip daddr 8.8.8.8 counter
nft add rule filter output tcp dport ssh counter
nft insert rule filter output ip daddr 192.168.1.1 counter

# 列出规则
nft list table filter

# 删除规则
nft list table filter -a # 查询handle是多少
nft delete rule filter output handle 5

# 删除链中所有规则
nft delete rule filter output

# 删除表中所有规则
nft flush table filter
```

## 负载均衡

负载均衡是一种将工作负载（如网络流量、请求、任务等）分配到多个资源上的技术。它旨在提高系统的性能、可靠性和可扩展性。

在负载均衡中，当有多个资源可用时，负载均衡器会根据一定的算法和策略将工作负载分发到这些资源上，以使每个资源都能够平衡地处理工作负载。这样可以避免某个资源过载而导致性能下降，同时提高整体系统的吞吐量和响应能力。

负载均衡可以应用于各种场景，包括网络负载均衡、服务器负载均衡、数据库负载均衡等。常见的负载均衡算法包括轮询、加权轮询、最少连接等，而负载均衡策略可以根据实际需求选择，如基于性能、基于会话等。

负载均衡的好处包括提高系统的可用性和可靠性，提升用户体验，提高系统的扩展性和弹性，避免单点故障等。它是构建高性能和高可用性系统的重要组成部分。

```
LVS（Linux Virtual Server）是一个虚拟的四层负载均衡集群系统，它在网络协议栈的传输层（第四层）上进行操作和转发。

LVS负载均衡系统工作在传输层，主要基于IP和端口信息来进行负载均衡。它使用IPVS内核模块来实现负载均衡功能。IPVS在Linux内核中作为一个模块存在，可以通过配置和管理VIP（虚拟IP）和RIP（真实IP）的映射关系，以及负载均衡的算法和策略。

传输层负载均衡主要关注连接的管理和转发，而不涉及应用层的具体协议。通过在传输层上进行负载均衡，LVS可以实现高性能的请求转发，同时具有较低的延迟和较高的吞吐量。此外，四层负载均衡还具有通用性，适用于各种应用和协议，如HTTP、TCP、UDP等。

需要注意的是，LVS也支持七层（应用层）负载均衡，即通过深度解析应用层协议来进行负载均衡。这种方式可以实现更精细的负载均衡策略，但相对于四层负载均衡来说，七层负载均衡的处理开销更大。因此，在性能要求较高的场景下，四层负载均衡通常更常见和推荐。
```

### lvs

`Linux Virtual Server` (`lvs`) 是`Linux`内核自带的负载均衡器，也是目前性能最好的软件负载均衡器之一。`lvs`包括`ipvs`内核模块和`ipvsadm`用户空间命令行工具两部分。

在`lvs`中，节点分为`Director Server`和`Real Server`两个角色，其中`Director Server`是负载均衡器所在节点，而`Real Server`则是后端服务节点。当用户的请求到达`Director Server`时，内核`netfilter`机制的`PREROUTING`链会将发往本地`IP`的包转发给`INPUT`链（也就是`ipvs`的工作链），在`INPUT`链上，`ipvs`根据用户定义的规则对数据包进行处理（如修改目的IP和端口等），并把新的包发送到`POSTROUTING`链，进而再转发给`Real Server`。

### 转发模式

```
在负载均衡中有链的概念，在配置iptables时，也可以使用类似的链的概念。然而，iptables中的链与负载均衡调度器中的链有一些不同之处。

1. 功能不同：负载均衡调度器中的链主要用于实现负载均衡和流量转发的功能。而在iptables中，链用于实现数据包过滤、网络地址转换（NAT）、网络地址端口转换（NAPT）等功能。

2. 规则集不同：负载均衡调度器中的链的规则集主要用于决定数据包的转发路径和目标服务器。而在iptables中，链的规则集用于匹配和处理数据包，决定是否接受、拒绝或修改数据包。

3. 执行顺序不同：在负载均衡调度器中，数据包经过链的顺序是有固定的流程的，一般是PREROUTING、INPUT、FORWARD、OUTPUT和POSTROUTING。而在iptables中，可以根据需要自定义链的顺序，并且可以根据匹配规则进行跳转。

虽然负载均衡调度器和iptables都使用了链的概念，但它们的功能和用途有所不同。负载均衡调度器中的链主要用于实现负载均衡和流量转发，而iptables中的链用于实现数据包过滤和网络地址转换等功能。
```

重要：[你真的掌握lvs工作原理吗？ (qq.com)](https://mp.weixin.qq.com/s?\_\_biz=MzAxMTkwODIyNA==\&mid=2247493477\&idx=1\&sn=f4833c3a5a82cffb0798cfc986afc911\&source=41#wechat\_redirect)

#### NAT

`NAT`模式通过修改数据包的目的IP和目的端口来将包转发给`Real Server`。它的特点包括

* `Director Server`必须作为`Real Server`的网关，并且它们必须处于同一个网段内
* 不需要`Real Server`做任何特殊配置
* 支持端口映射
* 请求和响应都需要经过`Director Server`，易称为性能瓶颈

![LVS NAT](https://tonydeng.gitbooks.io/sdn/content/linux/images/lvs-nat.png)

```
在NAT转发模式中，以下术语有特定的含义：

1. VIP（Virtual IP）：虚拟IP，是一个在网络中用于标识负载均衡服务的IP地址。当客户端发送请求时，请求会被转发到VIP上，然后根据负载均衡算法选择一个后端服务器来处理请求。VIP通常是一个虚拟的地址，不直接与任何实际的主机关联。

2. CIP（Client IP）：客户端IP，是指发起请求的客户端的真实IP地址。在NAT转发模式中，客户端的请求经过负载均衡器后，负载均衡器会将CIP替换为VIP，并将请求转发给后端服务器。

3. IPVS（IP Virtual Server）：IP虚拟服务器，是一个用于实现负载均衡的内核模块。IPVS可以在Linux操作系统上实现负载均衡，它通过管理VIP和后端服务器的映射关系，将请求转发到合适的后端服务器上。

4. RIP（Real IP）：真实IP，指后端服务器的实际IP地址。当负载均衡器收到请求后，根据负载均衡算法选择一个后端服务器，并将请求转发给该服务器的RIP。

综上所述，VIP是负载均衡服务的虚拟IP地址，CIP是客户端的真实IP地址，IPVS是实现负载均衡的内核模块，RIP是后端服务器的真实IP地址。在NAT转发模式中，负载均衡器通过将CIP替换为VIP，并将请求转发给合适的后端服务器（RIP），实现负载均衡功能。
```

#### DR

`DR`（`Direct Route`）模式通过修改数据包的目的`MAC`地址将包转发给`Real Server`。它的特点包括

* 需要在`Real Server`的`lo`上配置`vip`，并配置`arp_ignore`和`arp_announce`忽略对`vip`的`ARP`解析请求
* `Director Server`和`Real Server`必须在同一个物理网络内，二层可达
* 虽然所有请求包都会经过`Director Server`，但响应报文不经过，有性能上的优势

![LVS DR](https://tonydeng.gitbooks.io/sdn/content/linux/images/lvs-dr.png)

#### TUN

TUN模式通过将数据包封装在另一个IP包中（源地址为`DIP`，目的为`RIP`）将包转发给`Real Server`。它的特点包括

* `Real Server`需要在`lo`上配置`vip`，但不需要`Director Server`作为网关
* 不支持端口映射

![img](https://tonydeng.gitbooks.io/sdn/content/linux/images/lvs-tun.png)

#### FULLNAT

[lvs负载均衡fullnat](http://wangxuemin.github.io/2015/07/26/lvs%20%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1fullnat%20%E6%A8%A1%E5%BC%8Fclientip%20%E6%80%8E%E6%A0%B7%E4%BC%A0%E9%80%92%E7%BB%99%20realserver/)

`FULLNAT`是阿里在NAT基础上增加的一个新转发模式，通过引入`local IP`（`CIP-VIP转换为LIP->RIP`，而`LIP`和`RIP`均为`IDC内网IP`）使得物理网络可以跨越不同`vlan`，代码维护在[alibaba/LVS](https://github.com/alibaba/LVS)上面。其特点是

* 物理网络仅要求三层可达
* `Real Server`不需要任何特殊配置
* `SYNPROXY`防止`synflooding`攻击
* 未进入内核主线，维护复杂

![LVS FULLNAT](https://tonydeng.gitbooks.io/sdn/content/linux/images/lvs-fullnat.png)

### 调度算法

* 轮叫调度（`Round-Robin Scheduling`）
* 加权轮叫调度（`Weighted Round-Robin Scheduling`）
* 最小连接调度（`Least-Connection Scheduling`）
* 加权最小连接调度（`Weighted Least-Connection Scheduling`）
* 基于局部性的最少链接（`Locality-Based Least Connections Scheduling`）
* 带复制的基于局部性最少链接（`Locality-Based Least Connections with Replication Scheduling`）
* 目标地址散列调度（`Destination Hashing Scheduling`）
* 源地址散列调度（`Source Hashing Scheduling`）
* 最短预期延时调度（`Shortest Expected Delay Scheduling`）
* 不排队调度（`Never Queue Scheduling`）

### lvs配置示例

安装`ipvs`包并开启`ip`转发

```sh
yum -y install ipvsadm keepalived
sysctl -w net.ipv4.ip_forward=1


1. `yum -y install ipvsadm keepalived`：这个命令使用yum包管理器在系统上安装ipvsadm和keepalived软件包。ipvsadm是用于配置和管理IPVS负载均衡的工具，而keepalived是一个用于实现高可用性的软件，可以与IPVS结合使用。

2. `sysctl -w net.ipv4.ip_forward=1`：这个命令通过sysctl工具修改系统内核参数，将net.ipv4.ip_forward设置为1。这个参数表示启用IP转发功能，允许Linux系统作为一个路由器转发IP数据包。在负载均衡场景中，这个参数需要启用，以便负载均衡器可以将请求转发给后端服务器。
```

修改`/etc/keepalived/keepalived.conf`，增加`vip`和`lvs`的配置

```
vrrp_instance VI_3 {
    state MASTER   # 另一节点为BACKUP
    interface eth0
    virtual_router_id 11
    priority 100   # 另一节点为50
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass PASSWORD
    }

    track_script {
        chk_http_port
    }

    virtual_ipaddress {
        192.168.0.100
    }
}

virtual_server 192.168.0.100 9696 {
    delay_loop 30
    lb_algo rr
    lb_kind DR
    persistence_timeout 30
    protocol TCP

    real_server 192.168.0.101 9696 {
        weight 3
        TCP_CHECK {
            connect_timeout 10
            nb_get_retry 3
            delay_before_retry 3
            connect_port 9696
        }
    }

    real_server 192.168.0.102 9696 {
        weight 3
        TCP_CHECK {
            connect_timeout 10
            nb_get_retry 3
            delay_before_retry 3
            connect_port 9696
        }
    }
}
```

```
这是一个Keepalived配置文件的示例，用于实现高可用性负载均衡。

在这个示例中，配置了一个虚拟路由器实例（vrrp_instance）和一个虚拟服务器（virtual_server）。以下是配置文件中的关键部分：

1. vrrp_instance：
   - state：指定节点的状态，这里设置为MASTER，另一节点应设置为BACKUP。
   - interface：指定用于通信的网络接口。
   - virtual_router_id：指定虚拟路由器的ID，用于标识同一组节点。
   - priority：指定节点的优先级，用于选举MASTER节点，另一节点的优先级应设置为较低的值。
   - advert_int：指定广告间隔，即节点发送VRRP广告的时间间隔。
   - authentication：指定认证配置，这里使用密码认证方式。
   - track_script：指定要跟踪的脚本，用于检测节点的健康状态。
   - virtual_ipaddress：指定虚拟IP地址，这里设置为192.168.0.100。

2. virtual_server：
   - delay_loop：指定循环延迟时间。
   - lb_algo：指定负载均衡算法，这里使用了轮询（rr）算法。
   - lb_kind：指定负载均衡类型，这里使用了直接路由
```

重启`keepalived`：

```sh
systemctl reload keepalived
```

最后在`neutron-server`所在机器上为`lo`配置`vip`，并抑制`ARP`响应：

```sh
vip=192.168.0.100
ifconfig lo:1 ${vip} broadcast ${vip} netmask 255.255.255.255
route add -host ${vip} dev lo:1
echo "1" >/proc/sys/net/ipv4/conf/lo/arp_ignore
echo "2" >/proc/sys/net/ipv4/conf/lo/arp_announce
echo "1" >/proc/sys/net/ipv4/conf/all/arp_ignore
echo "2" >/proc/sys/net/ipv4/conf/all/arp_announce
```

```
首先，将VIP地址设置为192.168.0.100。
然后，使用ifconfig命令创建一个名为lo:1的虚拟网络接口，并将VIP地址分配给该接口。同时设置广播地址和子网掩码。
接下来，使用route add命令将VIP地址添加到lo:1接口上。
然后，通过修改/proc/sys/net/ipv4/conf/lo/arp_ignore和/proc/sys/net/ipv4/conf/lo/arp_announce文件的值，设置lo接口对ARP请求的处理方式。
最后，通过修改/proc/sys/net/ipv4/conf/all/arp_ignore和/proc/sys/net/ipv4/conf/all/arp_announce文件的值，设置所有网络接口对ARP请求的处理方式。

具体含义如下：
- arp_ignore的值为1，表示忽略所有接收到的ARP请求。
- arp_announce的值为2，表示在发送ARP请求时，使用接口上的IP地址作为源地址。

这些设置通常用于实现负载均衡、故障转移或高可用性网络配置。
```

### LVS缺点

* `Keepalived`主备模式设备利用率低；不能横向扩展；`VRRP`协议，有脑裂的风险。
* `ECMP`的方式需要了解动态路由协议，`LVS`和交换机均需要较复杂配置；交换机的`HASH`算法一般比较简单，增加删除节点会造成`HASH`重分布，可能导致当前`TCP`连接全部中断；部分交换机的`ECMP`在处理分片包时会有`BUG`。

### Haproxy

`Haproxy`也是`Linux`最常用的负载均衡软件之一，兼具性能和功能的组合，同时支持`TCP`和`HTTP`负载均衡。

配置和使用方法请见[官网](http://www.haproxy.org/)。

### Nginx

`Nginx`也是`Linux`最常用的负载均衡软件之一，常用作反向代理和`HTTP`负载均衡（当然也支持`TCP`和`UDP`负载均衡）。

配置和使用方法请见[官网](https://nginx.org/en/)。

### 自研负载均衡

#### Google Maglev

[Maglev](https://research.google.com/pubs/pub44824.html)是`Google`自研的负载均衡方案，在2008年就已经开始用于生产环境。`Maglev`安装后不需要预热5秒内就能处理`每秒100万次请求`。谷歌的性能基准测试中，`Maglev`实例运行在一个8核CPU下，网络吞吐率上限为`12M PPS（数据包每秒）`。如果`Maglev`使用`Linux`内核网络堆栈则速度会慢下来，吞吐率小于`4M PPS`。

![google maglev](https://tonydeng.gitbooks.io/sdn/content/linux/images/maglev.png)

* 路由器`ECMP` (`Equal Cost Multipath`) 转发包到`Maglev`（而不是传统的主从结构)
* `Kernel Bypass`, `CPU`绑定，共享内存
* 一致性哈希保证连接不中断

#### UCloud Vortex

`Vortex`参考了`Maglev`，大致的架构和实现跟`Maglev`类似：

* `ECMP`实现集群的负载均衡
* 一致性哈希保证连接不中断
  * 即使是不同的`Vortex`服务器收到了数据包，仍然能够将该数据包转发到同一台后端服务器
  * 后端服务器变化时，通过连接追踪机制保证当前活动连接的数据包被送往之前选择的服务器，而所有新建连接则会在变化后的服务器集群中进行负载分担
* DPDK提升单机性能 (14M PPS，10G, 64字节线速)
  * 通过`RSS`直接将网卡队列和`CPU Core`绑定，消除线程的上下文切换带来的开销
  * `Vortex`线程间采用高并发无锁的消息队列通信
* `DR`模式避免额外开销

## 流量控制

流量控制（`Traffic Control`， `tc`）是`Linux`内核提供的流量限速、整形和策略控制机制。它以`qdisc-class-filter`的树形结构来实现对流量的分层控制 ：`tc`最佳的参考就是[Linux Traffic Control HOWTO](http://www.tldp.org/HOWTO/Traffic-Control-HOWTO/)，详细介绍了tc的原理和使用方法。

### 基本组成

tc由`qdisc`、`fitler`和`class`三部分组成：

* `qdisc`通过队列将数据包缓存起来，用来控制网络收发的速度
* `class`用来表示控制策略
* `filter`用来将数据包划分到具体的控制策略中

### qdisc

`qdisc`通过队列将数据包缓存起来，用来控制网络收发的速度。实际上，每个网卡都有一个关联的`qdisc`。它包括以下几种：

* [无分类qdisc（只能应用于root队列）](http://tldp.org/HOWTO/Traffic-Control-HOWTO/classless-qdiscs.html)
  1. `[p|b]fifo`：简单先进先出
  2. `pfifo_fast`：根据数据包的`tos`将队列划分到3个`band`，每个`band`内部先进先出
  3. `red`：`Random Early Detection`，带带宽接近限制时随机丢包，适合高带宽应用
  4. `sfq`：`Stochastic Fairness Queueing`，按照会话对流量排序并循环发送每个会话的数据包
  5. `tbf`：`Token Bucket Filter`，只允许以不超过事先设定的速率到来的数据包通过 , 但可能允许短暂突发流量朝过设定值
*   [有分类qdisc（可以包括多个队列）](http://tldp.org/HOWTO/Traffic-Control-HOWTO/classful-qdiscs.html)

    1. `cbq`：`Class Based Queueing`，借助`EWMA`(`exponential weighted moving average`, 指数加权移动均值 ) 算法确认链路的闲置时间足够长 , 以达到降低链路实际带宽的目的。如果发生越限 ,`CBQ` 就会禁止发包一段时间。
    2. `htb`：`Hierarchy Token Bucket`，在`tbf`的基础上增加了分层
    3. `prio`：分类优先算法并不进行整形 , 它仅仅根据你配置的过滤器把流量进一步细分。缺省会自动创建三个`FIFO`类。

    注意，**一般说到qdisc都是指egress qdisc**。每块网卡实际上还可以添加一个`ingress qdisc`，不过它有诸多的限制

    * `ingress qdisc`不能包含子类，而只能作过滤
    * `ingress qdisc`只能用于简单的整形

    如果相对`ingress`方向作流量控制的话，可以借助[ifb（ Intermediate Functional Block）](https://wiki.linuxfoundation.org/networking/ifb)内核模块。因为流入网络接口的流量是无法直接控制的，那么就需要把流入的包导入（通过 `tc action`）到一个中间的队列，该队列在 ifb 设备上，然后让这些包重走 `tc` 层，最后流入的包再重新入栈，流出的包重新出栈。

![Intermediate Funcational Block](https://github.com/tonydeng/sdn-handbook/raw/master/linux/images/ifb.jpeg)

### filter

`filter`用来将数据包划分到具体的控制策略中，包括以下几种：

* `u32`：根据协议、`IP`、端口等过滤数据包
* `fwmark`：根据`iptables MARK`来过滤数据包
* `tos`：根据`tos`字段过滤数据包

### class

`class`用来表示控制策略，只用于有分类的`qdisc`上。每个`class`要么包含多个子类，要么只包含一个子`qdisc`。当然，每个`class`还包括一些列的`filter`，控制数据包流向不同的子类，或者是直接丢掉。

### htb示例

![Simplified Linux Traffic Control Scenario with HTB](https://github.com/tonydeng/sdn-handbook/raw/master/linux/images/htb-class.png)

```sh
# add qdisc
tc qdisc add dev eth0 root handle 1: htb default 2 r2q 100
# add default class
tc class add dev eth0 parent 1:0 classid 1:1 htb rate 1000mbit ceil 1000mbit
tc class add dev eth0 parent 1:1 classid 1:2 htb prio 5 rate 1000mbit ceil 1000mbit
tc qdisc add dev eth0 parent 1:2 handle 2: pfifo limit 500
# add default filter
tc filter add dev eth0 parent 1:0 prio 5 protocol ip u32
tc filter add dev eth0 parent 1:0 prio 5 handle 3: protocol ip u32 divisor 256
tc filter add dev eth0 parent 1:0 prio 5 protocol ip u32 ht 800:: match ip src 192.168.0.0/16 hashkey mask 0x000000ff at 12 link 3:

# add egress rules for 192.168.0.9
tc class add dev eth0 parent 1:1 classid 1:9 htb prio 5 rate 3mbit ceil 3mbit
tc qdisc add dev eth0 parent 1:9 handle 9: pfifo limit 500
tc filter add dev eth0 parent 1: protocol ip prio 5 u32 ht 3:9: match ip src "192.168.0.9" flowid 1:9
```

```sh
段代码是用于配置Linux系统上的流量控制（Traffic Control）规则。具体解释如下：

1. 第一行：添加一个队列规则（qdisc）到名为eth0的网络接口上。该规则使用htb算法进行流量控制，设置默认类别为2，队列长度为100。

2. 第二行：添加一个默认类别（class）规则到eth0接口上。该规则的父类别为1:0，类别标识为1:1，使用htb算法进行流量控制，设置速率为1000mbit，上限为1000mbit。

3. 第三行：添加一个子类别规则到1:1类别上。该规则的父类别为1:1，类别标识为1:2，使用htb算法进行流量控制，设置优先级为5，速率为1000mbit，上限为1000mbit。

4. 第四行：添加一个队列规则到1:2类别上。该规则使用pfifo算法进行队列管理，设置队列长度为500。

5. 第五行：添加一个过滤器规则到1:0类别上。该规则的优先级为5，协议为IP，使用u32匹配规则。

6. 第六行：添加一个过滤器规则到1:0类别上。该规则的优先级为5，使用handle为3的匹配规则。

7. 第七行：添加一个过滤器规则到1:0类别上。该规则的优先级为5，协议为IP，使用u32匹配规则，匹配源IP地址为192.168.0.0/16。

8. 第八行：为IP地址为192.168.0.9的流量添加一个子类别规则。该规则的父类别为1:1，类别标识为1:9，使用htb算法进行流量控制，设置优先级为5，速率为3mbit，上限为3mbit。

9. 第九行：为1:9类别添加一个队列规则。该规则使用pfifo算法进行队列管理，设置队列长度为500。

10. 第十行：为IP地址为192.168.0.9的流量添加一个过滤器规则。该规则的协议为IP，优先级为5，匹配源IP地址为192.168.0.9，并将匹配的流量流向1:9类别。
```

### ifb示例

```sh
# init ifb
modprobe ifb numifbs=1
ip link set ifb0 up
#  redirect ingress to ifb0
tc qdisc add dev eth0 ingress handle ffff:
tc filter add dev eth0 parent ffff: protocol ip prio 0 u32 match u32 0 0 flowid ffff: action mirred egress redirect dev ifb0
# add qdisc
tc qdisc add dev ifb0 root handle 1: htb default 2 r2q 100
# add default class
tc class add dev ifb0 parent 1:0 classid 1:1 htb rate 1000mbit ceil 1000mbit
tc class add dev ifb0 parent 1:1 classid 1:2 htb prio 5 rate 1000mbit ceil 1000mbit
tc qdisc add dev ifb0 parent 1:2 handle 2: pfifo limit 500
# add default filter
tc filter add dev ifb0 parent 1:0 prio 5 protocol ip u32
tc filter add dev ifb0 parent 1:0 prio 5 handle 4: protocol ip u32 divisor 256
tc filter add dev ifb0 parent 1:0 prio 5 protocol ip u32 ht 800:: match ip dst 192.168.0.0/16 hashkey mask 0x000000ff at 16 link 4:

# add ingress rules for 192.168.0.9
tc class add dev ifb0 parent 1:1 classid 1:9 htb prio 5 rate 3mbit ceil 3mbit
tc qdisc add dev ifb0 parent 1:9 handle 9: pfifo limit 500
tc filter add dev ifb0 parent 1: protocol ip prio 5 u32 ht 4:9: match ip dst "192.168.0.9" flowid 1:9
```

```sh
这段代码是用于配置Linux系统上的流量控制规则，但是这次是针对入站流量（ingress）进行控制。具体解释如下：

1. 第一行：加载ifb模块，并创建一个ifb接口。numifbs=1表示创建一个ifb接口。

2. 第二行：启用ifb0接口。

3. 第三行：将eth0接口的入站流量重定向到ifb0接口。

4. 第四行：为ifb0接口添加一个队列规则，使用htb算法进行流量控制，设置默认类别为2，队列长度为100。

5. 第五行：为ifb0接口添加一个默认类别规则，父类别为1:0，类别标识为1:1，使用htb算法进行流量控制，设置速率为1000mbit，上限为1000mbit。

6. 第六行：为1:1类别添加一个子类别规则，类别标识为1:2，使用htb算法进行流量控制，设置优先级为5，速率为1000mbit，上限为1000mbit。

7. 第七行：为1:2类别添加一个队列规则，使用pfifo算法进行队列管理，设置队列长度为500。

8. 第八行：为ifb0接口添加一个过滤器规则，优先级为5，协议为IP，使用u32匹配规则。

9. 第九行：为ifb0接口添加一个过滤器规则，优先级为5，使用handle为4的匹配规则。

10. 第十行：为ifb0接口添加一个过滤器规则，优先级为5，协议为IP，使用u32匹配规则，匹配目标IP地址为192.168.0.0/16。

11. 第十一行：为目标IP地址为192.168.0.9的入站流量添加一个子类别规则，父类别为1:1，类别标识为1:9，使用htb算法进行流量控制，设置优先级为5，速率为3mbit，上限为3mbit。

12. 第十二行：为1:9类别添加一个队列规则，使用pfifo算法进行队列管理，设置队列长度为500。

13. 第十三行：为目标IP地址为192.168.0.9的入站流量添加一个过滤器规则，协议为IP，优先级为5，匹配目标IP地址为192.168.0.9，并将匹配的流量流向1:9类别。
```

### 参考文档

http://www.tldp.org/HOWTO/Traffic-Control-HOWTO/

https://wiki.linuxfoundation.org/networking/ifb

http://blog.csdn.net/dog250/article/details/40483627

http://blog.csdn.net/dog250/article/details/40680765

## SR-IOV

SR-IOV是Single Root I/O Virtualization的缩写。在虚拟机中，一切皆虚拟。比如网卡，虚拟机看来好像有一个真实网卡，但是这个网卡是宿主机虚拟出来的硬件，也就是一堆软件代码而已，没有真实硬件。

虚拟虽然万能，但是很显然，这一堆代码是需要CPU去执行的！所以，虚拟设备的性能会随着宿主机的性能而改变，另外宿主机由于需要进行数据处理，时延等也产生了。

我们之前也提到过，VT-D这个功能可以将物理的PCI-e设备直接分配给虚拟机，让虚拟机直接控制硬件，那么就可以避开上述的问题。但是，虚拟机会独占这个直通的PCI-e设备，一台宿主机可能有成百上千个虚拟机，如果每个虚拟机都给直通一个PCI-e设备，比如网卡，那么该造什么样的主机？拥有100个物理网卡的主机？想象一下也是不太可能。

为了解决这个问题，Intel提出来SR-IOV这个东西。SR-IOV最初应用在网卡上。简单的说，就是一个物理网卡可以虚拟出来多个轻量化的PCI-e物理设备，从而可以分配给虚拟机使用。大概的框图如下：

![img](https://pic2.zhimg.com/80/v2-02865d3c697f77ba4c8c251de49e6855\_720w.webp)

SR-IOV是虚拟化的一个重要功能。启用SR-IOV的这个功能，将大大减轻宿主机的CPU负荷，提高网络性能，降低网络时延等。

性能对比试验请参考这篇文章：[SR-IOV是什么？性能能好到什么程度？ - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/91197211)

***

`SR-IOV`（`Single Root I/O Virtualization`）是一个将`PCIe`共享给虚拟机的标准，通过为虚拟机提供独立的内存空间、中断、`DMA`流，来绕过`VMM`实现数据访问。SR-IOV基于两种`PCIe functions`：

```sh
PCIe（Peripheral Component Interconnect Express）是一种高速串行总线接口，用于连接计算机的主板和各种外部设备，如显卡、网卡、声卡等。它是传统PCI（Peripheral Component Interconnect）总线的升级版本，提供了更高的传输速度和更低的延迟。

PCIe采用了串行数据传输方式，相比并行传输的PCI总线，具有更高的带宽和更低的功耗。它的传输速度通常以“x1”、“x4”、“x8”和“x16”来表示，分别表示单通道、四通道、八通道和十六通道的带宽。例如，一个PCIe x16插槽可以提供最高16GB/s的传输速度。

PCIe还支持热插拔功能，可以在计算机运行时插拔外部设备，而无需重启系统。此外，PCIe还具有更好的可扩展性，可以通过插槽数量的增加来支持更多的外部设备。

总之，PCIe是一种高速、低延迟、可扩展的总线接口，广泛应用于现代计算机系统中。
```

* `PF` (`Physical Function`)： 包含完整的`PCIe`功能，包括`SR-IOV`的扩张能力，该功能用于`SR-IOV`的配置和管理。
* `VF` (`Virtual Function`)： 包含轻量级的`PCIe`功能。每一个`VF`有它自己独享的`PCI`配置区域，并且可能与其他`VF`共享着同一个物理资源。

下图位SR-IOV架构：

![SR-IOV Architecture](https://tonydeng.gitbooks.io/sdn/content/linux/images/sriovarchitecture.png)

### SR-IOV要求

* `CPU` 必须支持`IOMMU`（比如英特尔的`VT-d` 或者`AMD`的 `AMD-Vi`，`Power8` 处理器默认支持`IOMMU`）
* 固件`Firmware` 必须支持`IOMMU`
* `CPU` 根桥必须支持 `ACS` 或者`ACS`等价特性
* `PCIe` 设备必须支持`ACS` 或者`ACS`等价特性
* 建议根桥和`PCIe` 设备中间的所有`PCIe` 交换设备都支持ACS，如果某个`PCIe`交换设备不支持`ACS`，其后的所有`PCIe`设备只能共享某个`IOMMU` 组，所以只能分配给1台虚机。

```sh
这段文字提到了一些硬件要求和建议，这些要求和建议适用于使用虚拟化技术时的系统配置。
首先，CPU必须支持IOMMU（输入输出内存管理单元）技术，例如英特尔的VT-d或者AMD的AMD-Vi，Power8处理器默认支持IOMMU。IOMMU是一种硬件功能，它允许虚拟机直接访问物理设备，提高虚拟化性能和安全性。
其次，固件（Firmware）必须支持IOMMU。固件是计算机系统中的低级软件，负责初始化硬件并启动操作系统。固件需要支持IOMMU以确保虚拟机可以正确地使用IOMMU功能。
接下来，CPU根桥必须支持ACS（Access Control Services）或者ACS等价特性。ACS是一种PCIe（Peripheral Component Interconnect Express）设备的功能，它允许将一个PCIe设备的IOMMU组划分为多个独立的IOMMU组。CPU根桥支持ACS可以提供更好的虚拟化性能和资源隔离。
同样地，PCIe设备也必须支持ACS或者ACS等价特性。这些设备需要支持ACS以便能够在虚拟化环境中正确地划分IOMMU组。
最后，建议所有PCIe交换设备都支持ACS。如果某个PCIe交换设备不支持ACS，那么它之后的所有PCIe设备只能共享一个IOMMU组，这将限制虚拟机的分配能力，只能将这些设备分配给一个虚拟机。
总之，以上要求和建议是为了确保在虚拟化环境中能够正确地使用IOMMU功能，提高虚拟化性能和资源隔离。
```

### SR-IOV vs 其他虚拟化技术

SR-IOV（Single Root I/O Virtualization）和PCI（Peripheral Component Interconnect）透传（PCI passthrough）是两种不同的虚拟化技术，用于在虚拟化环境中提供对物理设备的访问。

SR-IOV是一种硬件虚拟化技术，它允许物理设备（如网络适配器或存储适配器）在虚拟化环境中创建虚拟功能（VF），每个VF都可以被分配给虚拟机。SR-IOV通过硬件的DMA（直接内存访问）功能，将数据直接从物理设备传输到虚拟机，绕过虚拟机管理程序（如Hypervisor），从而提供更低的延迟和更高的性能。

PCI透传是一种软件虚拟化技术，它允许将物理设备直接分配给虚拟机，使虚拟机能够直接控制和访问该设备。在PCI透传中，虚拟机可以完全绕过虚拟化层，直接与物理设备进行通信，从而获得接近于物理机的性能。

这两种技术可以进行比较，因为它们都提供了对物理设备的直接访问，以提高虚拟机的性能。然而，它们在实现方式和适用场景上存在一些不同之处。

SR-IOV适用于需要高性能网络或存储访问的场景，它可以提供更低的延迟和更高的吞吐量。它依赖于硬件支持，并需要在物理设备上启用SR-IOV功能。

PCI透传适用于需要对特定物理设备进行直接控制的场景，例如需要使用特定的硬件驱动程序或访问设备的专有功能。它可以提供更高的灵活性和兼容性，因为它不受硬件SR-IOV支持的限制。

总而言之，SR-IOV和PCI透传都是提高虚拟机性能的技术，但它们的实现方式和适用场景有所不同。选择哪种技术取决于具体的需求和硬件支持。

```sh
SR-IOV、PCI透传、Virtio和vhost-net是虚拟化领域中常见的技术，它们有以下区别：

1. SR-IOV（Single Root I/O Virtualization）是一种硬件虚拟化技术，它允许物理设备（如网络适配器或存储适配器）在虚拟化环境中创建虚拟功能（VF），每个VF都可以被分配给虚拟机。SR-IOV通过硬件的DMA（直接内存访问）功能，将数据直接从物理设备传输到虚拟机，绕过虚拟机管理程序（如Hypervisor），从而提供更低的延迟和更高的性能。

2. PCI透传（PCI passthrough）是一种软件虚拟化技术，它允许将物理设备直接分配给虚拟机，使虚拟机能够直接控制和访问该设备。在PCI透传中，虚拟机可以完全绕过虚拟化层，直接与物理设备进行通信，从而获得接近于物理机的性能。

3. Virtio是一种虚拟化I/O设备的标准接口，它提供了一组标准的设备驱动程序和设备模型，用于在虚拟机和宿主机之间进行高效的数据传输和通信。Virtio可以提供较高的性能和较低的延迟，并且具有良好的兼容性和可移植性。

4. vhost-net是一种在虚拟化环境中加速网络性能的技术，它通过将网络数据包的处理从虚拟机移动到宿主机的用户空间，减少了虚拟机和宿主机之间的数据复制和上下文切换，从而提高了网络性能。vhost-net通常与Virtio配合使用，通过使用Virtio接口在虚拟机和vhost-net之间进行通信，进一步提高了网络性能。

综上所述，SR-IOV和PCI透传是直接将物理设备分配给虚拟机的技术，而Virtio和vhost-net是通过标准接口和网络加速技术来提高虚拟机性能的技术。它们在实现方式、性能特点和适用场景上有所不同，选择适合的技术取决于具体的需求和环境。
```

![img](https://images0.cnblogs.com/blog2015/697113/201506/041757191911525.jpg)

![img](https://images0.cnblogs.com/blog2015/697113/201506/041757434104956.jpg)

虚拟化方式的比较：\[KVM 介绍（4）：I/O 设备直接分配和 SR-IOV [KVM PCI/PCIe Pass-Through SR-IOV\] - SammyLiu - 博客园 (cnblogs.com)](https://www.cnblogs.com/sammyliu/p/4548194.html)

```
KVM（Kernel-based Virtual Machine）是一种开源的虚拟化解决方案，它是基于Linux内核的虚拟化模块。KVM利用Linux内核的虚拟化功能，将Linux内核转变为一个虚拟化管理程序（Hypervisor），提供了对硬件资源的虚拟化和管理。

KVM允许在宿主机上创建多个虚拟机，每个虚拟机可以运行独立的操作系统和应用程序。KVM利用虚拟化扩展（如Intel VT或AMD-V）提供硬件辅助虚拟化，使虚拟机能够直接访问物理硬件资源，从而获得接近于原生性能的运行效果。

KVM提供了一组工具和API，用于管理和配置虚拟机，包括创建、启动、停止、暂停、迁移等操作。KVM支持多种虚拟机格式，如QEMU（Quick Emulator）和Open Virtualization Format（OVF），可以与其他虚拟化管理工具（如libvirt）集成使用。

KVM是一种类型-1 Hypervisor，也就是说它直接运行在硬件上，而不需要在操作系统上安装额外的虚拟化软件。这种类型的Hypervisor通常提供更高的性能和更低的延迟，因为它可以直接访问物理硬件资源。

总的来说，KVM是一种基于Linux内核的开源虚拟化解决方案，通过利用硬件虚拟化扩展，它提供了对硬件资源的虚拟化和管理，使用户能够在宿主机上创建和管理多个虚拟机。
```

KVM性能测试：[Kvm performance optimization for ubuntu (slideshare.net)](https://www.slideshare.net/janghoonsim/kvm-performance-optimization-for-ubuntu)

SR-IOV vs DPDK（\[KVM 介绍（4）：I/O 设备直接分配和 SR-IOV [KVM PCI/PCIe Pass-Through SR-IOV\] - SammyLiu - 博客园 (cnblogs.com)](https://www.cnblogs.com/sammyliu/p/4548194.html)）

![sdn-fundamentals-for-nfv-openstack-and-containers-red-hat-summit](https://tonydeng.gitbooks.io/sdn/content/linux/images/sdn-fundamentals-for-nfv-openstack-and-containers-red-hat-summit-2016-47-638.jpg)

![sdn-fundamentals-for-nfv-openstack-and-containers-red-hat-summit](https://tonydeng.gitbooks.io/sdn/content/linux/images/sdn-fundamentals-for-nfv-openstack-and-containers-red-hat-summit-2016-48-638.jpg)

![sdn-fundamentals-for-nfv-openstack-and-containers-red-hat-summit](https://tonydeng.gitbooks.io/sdn/content/linux/images/sdn-fundamentals-for-nfv-openstack-and-containers-red-hat-summit-2016-49-638.jpg)

### SR-IOV使用示例

开启`VF`：

```sh
modprobe -r igb
modprobe igb max_vfs=7
echo "options igb max_vfs=7" >>/etc/modprobe.d/igb.conf

1. `modprobe -r igb`：该命令会从内核中移除 igb 模块。igb 是一个驱动程序模块，用于支持 Intel Gigabit 以太网适配器。通过这个命令，可以卸载 igb 模块。

2. `modprobe igb max_vfs=7`：该命令会将 igb 模块重新加载到内核，并设置 `max_vfs` 参数为 7。`max_vfs` 是 igb 模块的一个参数，用于设置虚拟功能的最大数量。通过这个命令，可以重新加载 igb 模块并设置虚拟功能的最大数量为 7。

3. `echo "options igb max_vfs=7" >>/etc/modprobe.d/igb.conf`：该命令会将 `options igb max_vfs=7` 这一行添加到 `/etc/modprobe.d/igb.conf` 文件中。`/etc/modprobe.d/igb.conf` 是一个配置文件，用于设置 igb 模块的参数。通过这个命令，可以将设置 `max_vfs` 参数为 7 的配置添加到 igb 模块的配置文件中。

igb 模块是一个驱动程序模块，用于支持 Intel Gigabit 以太网适配器。它是 Linux 内核中的一个网络驱动程序，为 Intel Gigabit 以太网适配器提供了网络功能。igb 模块可以与多种型号的 Intel Gigabit 以太网适配器一起使用，包括 Intel 82575、82576、82580、I350 和 I210 等系列。它负责管理和控制与这些适配器相关的网络通信和功能。通过加载 igb 模块，系统可以识别和使用相应的 Intel Gigabit 以太网适配器，并进行网络连接和数据传输。
```

查找`Virtual Function`：

```sh
# lspci | grep 82576
0b:00.0 Ethernet controller: Intel Corporation 82576 Gigabit Network Connection (rev 01)
0b:00.1 Ethernet controller: Intel Corporation 82576 Gigabit Network Connection(rev 01)
0b:10.0 Ethernet controller: Intel Corporation 82576 Virtual Function (rev 01)
0b:10.1 Ethernet controller: Intel Corporation 82576 Virtual Function (rev 01)
0b:10.2 Ethernet controller: Intel Corporation 82576 Virtual Function (rev 01)
0b:10.3 Ethernet controller: Intel Corporation 82576 Virtual Function (rev 01)
0b:10.4 Ethernet controller: Intel Corporation 82576 Virtual Function (rev 01)
0b:10.5 Ethernet controller: Intel Corporation 82576 Virtual Function (rev 01)
0b:10.6 Ethernet controller: Intel Corporation 82576 Virtual Function (rev 01)
0b:10.7 Ethernet controller: Intel Corporation 82576 Virtual Function (rev 01)
0b:11.0 Ethernet controller: Intel Corporation 82576 Virtual Function (rev 01)
0b:11.1 Ethernet controller: Intel Corporation 82576 Virtual Function (rev 01)
0b:11.2 Ethernet controller: Intel Corporation 82576 Virtual Function (rev 01)
0b:11.3 Ethernet controller: Intel Corporation 82576 Virtual Function (rev 01)
0b:11.4 Ethernet controller: Intel Corporation 82576 Virtual Function (rev 01)
0b:11.5 Ethernet controller: Intel Corporation 82576 Virtual Function (rev 01)

# virsh nodedev-list | grep 0b
pci_0000_0b_00_0
pci_0000_0b_00_1
pci_0000_0b_10_0
pci_0000_0b_10_1
pci_0000_0b_10_2
pci_0000_0b_10_3
pci_0000_0b_10_4
pci_0000_0b_10_5
pci_0000_0b_10_6
pci_0000_0b_11_7
pci_0000_0b_11_1
pci_0000_0b_11_2
pci_0000_0b_11_3
pci_0000_0b_11_4
pci_0000_0b_11_5
```

```sh
系统中有一块 Intel 82576 Gigabit 网络适配器。该适配器具有一个物理设备和多个虚拟功能。

物理设备：
- 0b:00.0 Ethernet controller: Intel Corporation 82576 Gigabit Network Connection (rev 01)
- 0b:00.1 Ethernet controller: Intel Corporation 82576 Gigabit Network Connection (rev 01)

虚拟功能：
- 0b:10.0 Ethernet controller: Intel Corporation 82576 Virtual Function (rev 01)
- 0b:10.1 Ethernet controller: Intel Corporation 82576 Virtual Function (rev 01)
- 0b:10.2 Ethernet controller: Intel Corporation 82576 Virtual Function (rev 01)
- 0b:10.3 Ethernet controller: Intel Corporation 82576 Virtual Function (rev 01)
- 0b:10.4 Ethernet controller: Intel Corporation 82576 Virtual Function (rev 01)
- 0b:10.5 Ethernet controller: Intel Corporation 82576 Virtual Function (rev 01)
- 0b:10.6 Ethernet controller: Intel Corporation 82576 Virtual Function (rev 01)
- 0b:10.7 Ethernet controller: Intel Corporation 82576 Virtual Function (rev 01)
- 0b:11.0 Ethernet controller: Intel Corporation 82576 Virtual Function (rev 01)
- 0b:11.1 Ethernet controller: Intel Corporation 82576 Virtual Function (rev 01)
- 0b:11.2 Ethernet controller: Intel Corporation 82576 Virtual Function (rev 01)
- 0b:11.3 Ethernet controller: Intel Corporation 82576 Virtual Function (rev 01)
- 0b:11.4 Ethernet controller: Intel Corporation 82576 Virtual Function (rev 01)
- 0b:11.5 Ethernet controller: Intel Corporation 82576 Virtual Function (rev 01)

这些虚拟功能是通过物理设备创建的，它们可以被用于虚拟化和网络功能的分配。
```

````sh
$ virsh nodedev-dumpxml pci_0000_0b_00_0
<device>
   <name>pci_0000_0b_00_0</name>
   <parent>pci_0000_00_01_0</parent>
   <driver>
      <name>igb</name>
   </driver>
   <capability type='pci'>
      <domain>0</domain>
      <bus>11</bus>
      <slot>0</slot>
      <function>0</function>
      <product id='0x10c9'>82576 Gigabit Network Connection</product>
      <vendor id='0x8086'>Intel Corporation</vendor>
   </capability>
</device>


根据您提供的信息，通过运行 `virsh nodedev-dumpxml pci_0000_0b_00_0` 命令，您可以获取与 `pci_0000_0b_00_0` 相关的设备信息。以下是命令输出的 XML 格式的设备描述：

```xml
<device>
   <name>pci_0000_0b_00_0</name>
   <parent>pci_0000_00_01_0</parent>
   <driver>
      <name>igb</name>
   </driver>
   <capability type='pci'>
      <domain>0</domain>
      <bus>11</bus>
      <slot>0</slot>
      <function>0</function>
      <product id='0x10c9'>82576 Gigabit Network Connection</product>
      <vendor id='0x8086'>Intel Corporation</vendor>
   </capability>
</device>
```

根据上述输出，可以得出以下设备信息：

- 设备名称：pci_0000_0b_00_0
- 父设备：pci_0000_00_01_0
- 驱动程序：igb
- PCI 能力：
   - 域（Domain）：0
   - 总线（Bus）：11
   - 插槽（Slot）：0
   - 功能（Function）：0
   - 产品 ID：0x10c9（82576 Gigabit Network Connection）
   - 厂商 ID：0x8086（Intel Corporation）

这些信息描述了设备的层次结构、驱动程序和 PCI 能力，有助于了解和管理设备。
````

通过libvirt绑定到虚拟机

````sh
$ cat >/tmp/interface.xml <<EOF
<interface type='hostdev' managed='yes'>
     <source>
       <address type='pci' domain='0' bus='11' slot='16' function='0'/>
     </source>
</interface>
EOF
$ virsh attach-device MyGuest /tmp/interface. xml --live --config


这两条命令的作用是将一个 PCI 设备附加到名为 "MyGuest" 的虚拟机上。

第一条命令将以下内容写入名为 "/tmp/interface.xml" 的文件中：

```xml
<interface type='hostdev' managed='yes'>
     <source>
       <address type='pci' domain='0' bus='11' slot='16' function='0'/>
     </source>
</interface>
```

这个 XML 描述了要附加的 PCI 设备。它指定了设备类型为 "hostdev"，表示将主机设备直接附加到虚拟机。`<source>` 元素指定了设备的地址，其中 `type='pci'` 表示设备是一个 PCI 设备，`domain='0'`、`bus='11'`、`slot='16'` 和 `function='0'` 分别指定了设备的 PCI 地址。

第二条命令使用 `virsh attach-device` 命令将 "/tmp/interface.xml" 文件中描述的设备附加到名为 "MyGuest" 的虚拟机上。`--live` 参数表示设备将立即在运行中的虚拟机上生效，`--config` 参数表示设备将在虚拟机下次启动时仍然保持附加状态。

通过这两个命令，您可以将指定的 PCI 设备直接附加到虚拟机中，以便虚拟机可以使用该设备的功能。
````

当然也可以给网卡配置`MAC`地址和`VLAN`：

```xml
<interface type='hostdev' managed='yes'>
     <source>
       <address type='pci' domain='0' bus='11' slot='16' function='0'/>
     </source>
     <mac address='52:54:00:6d:90:02'>
     <vlan>
        <tag id='42'/>
     </vlan>
     <virtualport type='802.1Qbh'>
       <parameters profileid='finance'/>
     </virtualport>
   </interface>
    
    
这个 XML 描述了要附加的 PCI 设备以及一些附加的网络属性：

- `<interface>` 元素的 `type` 属性设置为 "hostdev"，表示将主机设备直接附加到虚拟机。
- `<source>` 元素指定了设备的地址，使用了 `type='pci'`，`domain='0'`，`bus='11'`，`slot='16'` 和 `function='0'`。
- `<mac>` 元素指定了虚拟机中附加设备的 MAC 地址。
- `<vlan>` 元素指定了一个 VLAN 标签，其中 `id='42'` 表示 VLAN ID 为 42。
- `<virtualport>` 元素指定了虚拟端口类型为 "802.1Qbh"，并使用 `<parameters>` 元素指定了一个名为 "finance" 的配置文件。

```

通过Qemu绑定到虚拟机

```sh
/usr/bin/qemu-kvm -name vdisk -enable-kvm -m 512 -smp 2 \
-hda /mnt/nfs/vdisk.img \
-monitor stdio \
-vnc 0.0.0.0:0 \
-device pci-assign,host=0b:00.0

这是一个运行 QEMU-KVM 的命令，用于创建一个虚拟机并将指定的 PCI 设备附加到虚拟机中。
命令中的选项和参数的含义如下：

- `-name vdisk`：设置虚拟机的名称为 "vdisk"。
- `-enable-kvm`：启用 KVM 加速。
- `-m 512`：分配给虚拟机的内存大小为 512MB。
- `-smp 2`：虚拟机的 CPU 核心数为 2。
- `-hda /mnt/nfs/vdisk.img`：指定虚拟机的硬盘镜像文件为 "/mnt/nfs/vdisk.img"。
- `-monitor stdio`：将监控控制台输出重定向到标准输入/输出。
- `-vnc 0.0.0.0:0`：启用 VNC 服务器，并监听所有网络接口的 5900 端口。
- `-device pci-assign,host=0b:00.0`：将 PCI 设备 "0b:00.0" 
```

### 优缺点

```sh
优点：
比直接分配更具可扩展性
通过IOMMU和功能隔离提供安全性
通过PF/VF概念实现控制平面分离
通过直接透传实现高数据包速率、低CPU负载和低延迟

缺点：
刚性：可组合性问题
控制平面透传，对硬件资源施加压力
部分PCIe配置空间直接映射自硬件
可扩展性有限（16位）
SR-IOV网卡将交换功能强制置于硬件中
所有交换功能都在硬件中或者没有交换功能
```

## VRF

`Linux`内核的`Virtual Routing and Forwarding` (`VRF`) 是由路由表和一组网络设备组成的路由实例。

```sh
VRF（Virtual Routing and Forwarding）是一种网络虚拟化技术，用于在单个物理网络基础设施上创建多个逻辑上隔离的虚拟路由实例。使用VRF可以提供以下优势和用途：

1. 逻辑隔离：VRF允许将网络划分为多个逻辑上隔离的虚拟路由域，每个VRF都有自己的路由表、转发规则和网络资源。这种逻辑隔离可以增加网络的安全性和可靠性。

2. 多租户支持：VRF使得在单个物理网络基础设施上支持多个租户成为可能。每个租户可以拥有自己的独立网络环境，而不会相互干扰。

3. 路由策略控制：VRF允许在每个VRF中定义不同的路由策略和转发规则。这样，可以根据需求为不同的应用、用户或部门定制网络策略，提供更灵活的网络管理和控制。

4. 简化网络架构：通过使用VRF，可以将多个独立的网络部署在单个物理基础设施上，从而简化网络架构和管理。这可以减少硬件成本、降低复杂性，并提高网络资源的利用率。

5. 提高网络性能：通过在VRF中实现路由隔离，可以避免不同VRF之间的冲突和干扰，提高网络性能和吞吐量。

总而言之，VRF提供了一种有效的方式来实现网络的逻辑隔离、多租户支持、路由策略控制和网络简化，从而满足不同应用场景下的需求，并提高网络的安全性、可靠性和性能。
```

### VRF安装

`Ubuntu`默认不包括`vrf`内核模块，需要额外安装:

```sh
apt-get install linux-headers-4.10.0-14-generic linux-image-extra-4.10.0-14-generic
reboot
apt-get install linux-image-extra-$(uname -r)
modprobe vrf


- `apt-get install linux-headers-4.10.0-14-generic linux-image-extra-4.10.0-14-generic`：使用apt-get命令安装指定版本的Linux内核头文件和额外的内核镜像。
- `reboot`：重新启动系统，以使新安装的内核生效。
- `apt-get install linux-image-extra-$(uname -r)`：使用apt-get命令安装与当前正在运行的内核版本相对应的额外内核镜像。
- `modprobe vrf`：加载vrf内核模块，vrf是一种虚拟路由和转发技术。

```

### VRF示例

```sh
# create vrf device
ip link add vrf-blue type vrf table 10
# 创建名为vrf-blue的VRF设备，并将其关联到表格10。
ip link set dev vrf-blue up
# 启用vrf-blue设备

# An l3mdev FIB rule directs lookups to the table associated with the device.
# A single l3mdev rule is sufficient for all VRFs.
# Prior to the v4.8 kernel iif and oif rules are needed for each VRF device:
ip ru add oif vrf-blue table 10
# 添加一个输出接口规则，将流量从vrf-blue设备发送到表格10。
ip ru add iif vrf-blue table 10
# 添加一个输入接口规则，将流量从vrf-blue设备接收到表格10。

#Set the default route for the table (and hence default route for the VRF).
ip route add table 10 unreachable default
# 将默认路由添加到表格10，以阻止任何流量通过VRF设备。

# 以下命令用于将L3接口绑定到VRF设备：
# Enslave L3 interfaces to a VRF device.
# Local and connected routes for enslaved devices are automatically moved to
# the table associated with VRF device. Any additional routes depending on
# the enslaved device are dropped and will need to be reinserted to the VRF
# FIB table following the enslavement.
ip link set dev eth1 master vrf-blue
# 将eth1接口绑定到vrf-blue设备

# The IPv6 sysctl option keep_addr_on_down can be enabled to keep IPv6 global
# addresses as VRF enslavement changes.
sysctl -w net.ipv6.conf.all.keep_addr_on_down=1

# Additional VRF routes are added to associated table.
ip route add table 10 ...
# 添加其他VRF路由到关联的表格10中
```

### 进程绑定VRF

`Linux`进程可以通过在`VRF`设备上监听`socket`来绑定VRF：

```c
setsockopt(sd, SOL_SOCKET, SO_BINDTODEVICE, dev, strlen(dev)+1);


使用`setsockopt`函数，可以将套接字绑定到指定的VRF设备。具体来说，通过以下参数调用`setsockopt`函数可以实现这一目的：

- `sd` 是套接字描述符，用于标识要设置选项的套接字。
- `SOL_SOCKET` 是选项级别，表示要设置的选项属于套接字级别的选项。
- `SO_BINDTODEVICE` 是选项名称，表示要绑定套接字到指定的网络设备。
- `dev` 是一个指向要绑定的VRF设备名称的字符串。
- `strlen(dev)+1` 是指定要绑定的VRF设备名称的长度，加1是为了包括字符串的终止符。

通过调用`setsockopt`函数并传递这些参数，可以将套接字绑定到指定的VRF设备。这样，进程可以在该VRF上监听套接字，并与该VRF相关的网络进行通信。
```

```sh
sysctl -w net.ipv4.tcp_l3mdev_accept=1
sysctl -w net.ipv4.udp_l3mdev_accept=1

在默认VRF上运行的TCP和UDP服务可以通过启用tcp_l3mdev_accept和udp_l3mdev_accept sysctl选项来在所有VRF域中工作。

- `tcp_l3mdev_accept`是一个sysctl选项，用于启用TCP服务在所有VRF域中接受数据包。
- `udp_l3mdev_accept`是一个sysctl选项，用于启用UDP服务在所有VRF域中接受数据包。

通过启用这些选项，TCP和UDP服务可以在默认VRF上接收来自所有VRF域的数据包。这意味着这些服务不会受到VRF的限制，能够与所有VRF域进行通信。
```

### VRF操作

#### 创建VRF

```bash
ip link add dev NAME type vrf table ID

这是一个用于在Linux系统中创建VRF设备的命令。

- `ip link add dev NAME`：创建一个名为`NAME`的网络设备。
- `type vrf`：指定创建的设备类型为VRF。
- `table ID`：将VRF设备关联到ID为`ID`的路由表。

通过执行这个命令，可以创建一个名为`NAME`的VRF设备，并将其关联到指定的路由表ID。这样，VRF设备就可以用于实现虚拟路由和转发功能。
```

#### 查询VRF列表

```bash
# ip -d link show type vrf
16: vrf-blue: <NOARP,MASTER,UP,LOWER_UP> mtu 65536 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether 9e:9c:8e:7b:32:a4 brd ff:ff:ff:ff:ff:ff promiscuity 0
    vrf table 10 addrgenmode eui64
    
    
    
这是一个使用`ip`命令来显示VRF设备详细信息的示例输出。

- `16: vrf-blue`：设备名称为`vrf-blue`，编号为16。
- `<NOARP,MASTER,UP,LOWER_UP>`：设备的状态标志，表示该设备没有ARP协议、是主设备、处于启用状态、链路已连接。
- `mtu 65536`：设备的最大传输单元大小为65536字节。
- `qdisc noqueue`：设备没有指定队列调度器。
- `state UP`：设备处于UP（启用）状态。
- `mode DEFAULT`：设备的模式为DEFAULT。
- `group default`：设备所属的默认组。
- `qlen 1000`：设备的队列长度为1000。
- `link/ether 9e:9c:8e:7b:32:a4`：设备的链路层地址为9e:9c:8e:7b:32:a4。
- `brd ff:ff:ff:ff:ff:ff`：设备的广播地址为ff:ff:ff:ff:ff:ff。
- `promiscuity 0`：设备的混杂模式为0，表示不启用混杂模式。
- `vrf table 10`：设备关联的路由表ID为10。
- `addrgenmode eui64`：设备的地址生成模式为eui64。

这个输出提供了关于VRF设备的详细信息，包括设备名称、状态、链路信息、路由表关联等。
```

#### 添加网卡到VRF

```bash
ip link set dev eth0 master vrf-blue

这是一个使用`ip link`命令将`eth0`接口绑定到`vrf-blue` VRF设备的示例命令。

`ip link set dev eth0 master vrf-blue`：将`eth0`接口设置为`vrf-blue` VRF设备的从属接口。

通过执行这个命令，可以将`eth0`接口绑定到`vrf-blue` VRF设备上。这样，`eth0`接口的网络流量将通过该VRF进行路由和转发。
```

#### 查询VRF邻接表和路由

```bash
ip neigh show vrf vrf-blue
ip addr show vrf vrf-blue
ip -br addr show vrf vrf-blue
ip route show vrf vrf-blue


这些是一些使用`ip`命令在指定的VRF上显示网络邻居、IP地址和路由表的示例命令。

- `ip neigh show vrf vrf-blue`：显示在`vrf-blue` VRF上的网络邻居信息。
- `ip addr show vrf vrf-blue`：显示在`vrf-blue` VRF上的IP地址信息。
- `ip -br addr show vrf vrf-blue`：以简洁的格式显示在`vrf-blue` VRF上的IP地址信息。
- `ip route show vrf vrf-blue`：显示在`vrf-blue` VRF上的路由表信息。

通过执行这些命令，可以查看指定VRF上的网络邻居、IP地址和路由表的详细信息。这对于诊断和调试VRF网络配置非常有用。
```

#### 从VRF中删除网卡

```bash
ip link set dev eth0 nomaster
这是一个使用`ip link`命令将`eth0`接口从VRF设备中解绑的示例命令。

`ip link set dev eth0 nomaster`：将`eth0`接口从任何VRF设备中解绑。

通过执行这个命令，可以将`eth0`接口从之前绑定的VRF设备中解除绑定。这样，`eth0`接口将不再受限于特定的VRF，并恢复为默认的网络设备。
```

### 参考文章

https://www.kernel.org/doc/Documentation/networking/vrf.txt

## eBPF

```
BPF（Berkeley Packet Filter）是一种虚拟机器指令集，最初用于网络数据包过滤。它可以用于在内核空间中执行特定的过滤操作，以便选择性地处理或丢弃网络数据包。BPF是一种简单而有效的过滤机制，广泛应用于网络设备和安全工具中。

eBPF（extended Berkeley Packet Filter）是对BPF的扩展，它可以在内核中执行更复杂的任务。eBPF提供了一种安全且高效的方式，允许用户编写和加载自定义的程序代码到内核中，以实现各种功能，如网络分析、性能调优、安全监控等。

与传统的BPF相比，eBPF具有更强大的功能和更广泛的应用领域。它可以在运行时动态加载和更新程序代码，使得内核能够灵活地适应不同的需求。eBPF还提供了一组丰富的系统调用和内核函数，使得用户能够更方便地与内核进行交互。

总的来说，BPF和eBPF是一种在内核空间中执行程序代码的机制，用于实现网络数据包过滤、性能调优、安全监控等功能。eBPF是对BPF的扩展，提供了更强大的功能和更广泛的应用领域。
```

`eBPF`（`extended Berkeley Packet Filter`）起源于`BPF`，它提供了内核的数据包过滤机制。

BPF的基本思想是对用户提供两种`SOCKET`选项：`SO_ATTACH_FILTER`和`SO_ATTACH_BPF`，允许用户在`sokcet`上添加自定义的`filter`，只有满足该`filter`指定条件的数据包才会上发到用户空间。`SO_ATTACH_FILTER`插入的是`cBPF`代码，`SO_ATTACH_BPF`插入的是`eBPF`代码。`eBPF`是对`cBPF`的增强，目前用户端的`tcpdump`等程序还是用的`cBPF`版本，其加载到内核中后会被内核自动的转变为`eBPF`。

Linux 3.15 开始引入 `eBPF`。其扩充了 `BPF` 的功能，丰富了指令集。它在内核提供了一个虚拟机，用户态将过滤规则以虚拟机指令的形式传递到内核，由内核根据这些指令来过滤网络数据包。

![EBPF](https://tonydeng.gitbooks.io/sdn/content/linux/images/ebpf.png)

`BPF`和`eBPF`的内核文档见[Documentation/networking/filter.txt](https://www.kernel.org/doc/Documentation/networking/filter.txt)。

### 使用场景

`eBPF`使用场景包括

* `XDP`
* 流量控制
* 防火墙
* 网络包跟踪
* 内核探针
* `cgroups`
* [bcc](https://tonydeng.gitbooks.io/sdn/content/linux/bpf/bcc.html)
* [bpftools](https://github.com/cloudflare/bpftools)

### BCC

`BPF Compiler Collection` (`BCC`)是基于`eBPF`的`Linux`内核分析、跟踪、网络监控工具。其源码存放于[iovisor/bcc](https://github.com/iovisor/bcc)。

BCC包括的一些工具：

![Linux bcc/BPF Tracing Tools](https://github.com/iovisor/bcc/raw/master/images/bcc\_tracing\_tools\_2016.png)

#### 安装BCC

`Ubuntu`：

```sh
echo "deb [trusted=yes] https://repo.iovisor.org/apt/xenial xenial-nightly main" | sudo tee /etc/apt/sources.list.d/iovisor.list
sudo apt-get update
sudo apt-get install -y bcc-tools libbcc-examples python-bcc
```

`CentOS`：

```sh
echo -e '[iovisor]\nbaseurl=https://repo.iovisor.org/yum/nightly/f23/$basearch\nenabled=1\ngpgcheck=0' | sudo tee /etc/yum.repos.d/iovisor.repo
yum install -y bcc-tools
```

安装完成后，`bcc`工具会放到`/usr/share/bcc/tools`目录中

```sh
$ ls /usr/share/bcc/tools
argdist       cachestat  ext4dist        hardirqs        offwaketime  softirqs    tcpconnect  vfscount
bashreadline  cachetop   ext4slower      killsnoop       old          solisten    tcpconnlat  vfsstat
biolatency    capable    filelife        llcstat         oomkill      sslsniff    tcplife     wakeuptime
biosnoop      cpudist    fileslower      mdflush         opensnoop    stackcount  tcpretrans  xfsdist
biotop        dcsnoop    filetop         memleak         pidpersec    stacksnoop  tcptop      xfsslower
bitesize      dcstat     funccount       mountsnoop      profile      statsnoop   tplist      zfsdist
btrfsdist     doc        funclatency     mysqld_qslower  runqlat      syncsnoop   trace       zfsslower
btrfsslower   execsnoop  gethostlatency  offcputime      slabratetop  tcpaccept   ttysnoop
```

#### 常用工具示例

**capable**

`capable`检查`Linux`进程的`security capabilities`：

```sh
$ capable
TIME      UID    PID    COMM             CAP  NAME                 AUDIT
22:11:23  114    2676   snmpd            12   CAP_NET_ADMIN        1
22:11:23  0      6990   run              24   CAP_SYS_RESOURCE     1
22:11:23  0      7003   chmod            3    CAP_FOWNER           1
22:11:23  0      7003   chmod            4    CAP_FSETID           1
22:11:23  0      7005   chmod            4    CAP_FSETID           1
22:11:23  0      7005   chmod            4    CAP_FSETID           1
22:11:23  0      7006   chown            4    CAP_FSETID           1
22:11:23  0      7006   chown            4    CAP_FSETID           1
22:11:23  0      6990   setuidgid        6    CAP_SETGID           1
22:11:23  0      6990   setuidgid        6    CAP_SETGID           1
22:11:23  0      6990   setuidgid        7    CAP_SETUID           1
22:11:24  0      7013   run              24   CAP_SYS_RESOURCE     1
22:11:24  0      7026   chmod            3    CAP_FOWNER           1
[...]
```

```
"$ capable"显示了系统中运行的进程及其拥有的权限（capabilities）。每一行都列出了进程的时间戳、用户ID、进程ID、进程名称、权限（CAP）和权限名称。

在Linux系统中，进程可以通过权限来执行特定的操作或访问特定的资源。这些权限称为capabilities。每个进程都可以拥有一组capabilities，这些capabilities决定了进程可以执行的操作范围。在给出的输出示例中，每一行都显示了进程的权限（CAP）和权限名称。例如，第一行显示了进程snmpd拥有CAP_NET_ADMIN权限，即具备管理网络的能力。

总之，"$ capable"输出显示了系统中运行的进程及其拥有的安全权限。
```

**tcpconnect**

`tcpconnect`检查活跃的`TCP`连接，并输出源和目的地址：

```sh
$ ./tcpconnect
 PID    COMM         IP SADDR            DADDR            DPORT
 2462   curl         4  192.168.1.99       74.125.23.138    80
```

**tcptop**

`tcptop`统计TCP发送和接受流量：

```sh
$ ./tcptop -C 1 3
Tracing... Output every 1 secs. Hit Ctrl-C to end

08:06:45 loadavg: 0.04 0.01 0.00 2/174 3099

PID    COMM         LADDR                 RADDR                  RX_KB  TX_KB
1740   sshd         192.168.1.99:22         192.168.0.29:60315         0      0

08:06:46 loadavg: 0.04 0.01 0.00 2/174 3099

PID    COMM         LADDR                 RADDR                  RX_KB  TX_KB
1740   sshd         192.168.1.99:22         192.168.0.29:60315         0      0

08:06:47 loadavg: 0.04 0.01 0.00 2/174 3099

PID    COMM         LADDR                 RADDR                  RX_KB  TX_KB
1740   sshd         192.168.1.99:22         192.168.0.29:60315         0      0
```

```sh
参数"-C 1 3"表示在输出结果中显示每秒钟的网络负载情况和TCP连接信息，并且每秒钟输出一次，持续3次。

具体解释如下：
- "-C"是一个选项参数，用于指定输出结果的格式或显示方式。
- "1"是一个参数值，表示每秒钟输出一次结果。
- "3"是另一个参数值，表示输出结果持续3次。

因此，命令的含义是要求输出结果以每秒钟一次的频率显示网络负载情况和TCP连接信息，并持续输出3次。这可以用于实时监测网络负载和连接活动的变化。

- "loadavg: 0.04 0.01 0.00 2/174 3099"：这是一个关于系统负载情况的指标，它显示了系统在最近的1分钟、5分钟和15分钟内的负载平均值。在这个例子中，负载平均值分别为0.04、0.01和0.00。后面的"2/174"表示当前有2个运行中的进程，总共有174个进程。最后的"3099"表示系统中的总进程数。

- "PID"、"COMM"、"LADDR"、"RADDR"、"RX_KB"和"TX_KB"：这些是每个TCP连接的信息字段。
  - "PID"：进程ID，表示与该连接相关的进程的唯一标识符。
  - "COMM"：进程名称，表示与该连接相关的进程的名称或命令。
  - "LADDR"：本地地址，表示本地主机的IP地址和端口号。
  - "RADDR"：远程地址，表示远程主机的IP地址和端口号。
  - "RX_KB"：接收的数据量，表示从远程主机接收到的数据量（以千字节为单位）。
  - "TX_KB"：发送的数据量，表示发送到远程主机的数据量（以千字节为单位）。

在给定的示例中，只有一个TCP连接被显示出来，即进程ID为1740的sshd进程。它正在通过本地地址192.168.1.99的端口22与远程地址192.168.0.29的端口60315进行通信。该连接的接收和发送数据量都为0。

这段输出结果可以用于监视系统的网络负载状况和TCP连接的活动情况，以便进行网络性能分析和故障排除。
```

### 扩展工具

于`eBPF`和`bcc`，可以很方便的扩展功能。`bcc`目前支持以下事件

* `kprobe__kernel_function_name` (`BPF.attach_kprobe()`)
* `kretprobe__kernel_function_name` (`BPF.attach_kretprobe()`)
* `TRACEPOINT_PROBE(category, event)`，支持的`event`列表参见`/sys/kernel/debug/tracing/events/category/event/format`
* `BPF.attach_uprobe()`和`BPF.attach_uretprobe()`
* 用户自定义探针(USDT) `USDT.enable_probe()`

```
eBPF（Extended Berkeley Packet Filter）和bcc（BPF Compiler Collection）是用于在Linux内核中进行动态追踪和性能分析的工具和技术。

在bcc中，可以通过BPF.attach_kprobe()和BPF.attach_kretprobe()来注册内核函数的前后钩子，以便在函数执行前后执行自定义的eBPF程序。

TRACEPOINT_PROBE(category, event)允许注册跟踪点（tracepoint），以便在特定事件发生时执行eBPF程序。可以通过/sys/kernel/debug/tracing/events/category/event/format来查看支持的事件列表。

BPF.attach_uprobe()和BPF.attach_uretprobe()用于在用户空间程序中注册函数的前后钩子，以便在函数执行前后执行自定义的eBPF程序。

用户自定义探针（USDT）是一种允许用户在应用程序中插入自定义的探测点的机制。可以通过USDT.enable_probe()来启用自定义探针，并在探测点处执行eBPF程序。

总的来说，通过这些事件和探针机制，eBPF和bcc可以方便地扩展功能，实现对内核和用户空间程序的动态追踪和性能分析。
```

### 简单示例

```python
#!/usr/bin/env python
from __future__ import print_function
from bcc import BPF

text='int kprobe__sys_sync(void *ctx) { bpf_trace_printk("Hello, World!\\n"); return 0; }'
prog="""
int hello(void *ctx) {
        bpf_trace_printk("Hello, World!\\n");
        return 0;
}
"""

b = BPF(text=prog)
b.attach_kprobe(event="sys_clone", fn_name="hello")

print("%-18s %-16s %-6s %s" % ("TIME(s)", "COMM", "PID", "MESSAGE"))
while True:
        try:
                (task, pid, cpu, flags, ts, msg) = b.trace_fields()
        except ValueError:
                continue
        print("%-18.9f %-16s %-6d %s" % (ts, task, pid, msg))
```

```sh
这段代码是一个使用Python和bcc库编写的程序，用于跟踪并打印出发生在系统中的特定事件的信息。

具体来说，它使用了bcc库来编写和加载eBPF程序，然后通过跟踪特定事件来捕获和打印相关信息。

以下是代码的主要部分：

1. 导入所需的模块和库：从__future__模块导入print_function函数，导入BPF类和其他必要的模块。

2. 定义eBPF程序：在text和prog变量中定义了两个eBPF程序。这些程序使用bpf_trace_printk函数打印出"Hello, World!"的消息。

3. 创建BPF对象并注册钩子：使用BPF类创建一个BPF对象，并将定义的eBPF程序加载到该对象中。然后使用attach_kprobe()方法将钩子注册到sys_clone事件上，以便在sys_clone事件发生时执行hello函数。

4. 打印跟踪信息：使用trace_fields()方法从BPF对象中获取跟踪信息，并以格式化的方式打印出时间戳、进程名称、进程ID和消息。

总的来说，这段代码的目的是通过eBPF程序和bcc库来跟踪系统中的sys_clone事件，并在每次事件发生时打印出相关信息。
```

更多的示例参考[bbc/docs/tutorial\_bcc\_python\_developer.md](https://github.com/iovisor/bcc/blob/master/docs/tutorial\_bcc\_python\_developer.md)。

### 用户自定义探针示例

```python
from __future__ import print_function
from bcc import BPF
from time import strftime
import ctypes as ct

# load BPF program
bpf_text = """
#include <uapi/linux/ptrace.h>

struct str_t {
    u64 pid;
    char str[80];
};

BPF_PERF_OUTPUT(events);

int printret(struct pt_regs *ctx) {
    struct str_t data  = {};
    u32 pid;
    if (!PT_REGS_RC(ctx))
        return 0;
    pid = bpf_get_current_pid_tgid();
    data.pid = pid;
    bpf_probe_read(&data.str, sizeof(data.str), (void *)PT_REGS_RC(ctx));
    events.perf_submit(ctx,&data,sizeof(data));

    return 0;
};
"""
STR_DATA = 80

class Data(ct.Structure):
    _fields_ = [
        ("pid", ct.c_ulonglong),
        ("str", ct.c_char * STR_DATA)
    ]

b = BPF(text=bpf_text)
b.attach_uretprobe(name="/bin/bash", sym="readline", fn_name="printret")

# header
print("%-9s %-6s %s" % ("TIME", "PID", "COMMAND"))

def print_event(cpu, data, size):
    event = ct.cast(data, ct.POINTER(Data)).contents
    print("%-9s %-6d %s" % (strftime("%H:%M:%S"), event.pid, event.str))

b["events"].open_perf_buffer(print_event)
while 1:
    b.kprobe_poll()
```

```sh
这段代码使用了Python的`bcc`库来进行系统调用跟踪。具体解释如下：

1. `from __future__ import print_function`：这行代码是为了确保在Python 2.x版本中使用`print`函数而不是`print`语句。在Python 3.x中，`print`函数是默认的输出方式。

2. `from bcc import BPF`：导入`bcc`库，这是一个用于编写和加载eBPF程序的Python库。

3. `from time import strftime`：导入`strftime`函数，用于格式化时间戳。

4. `import ctypes as ct`：导入`ctypes`库，用于处理C语言数据类型。

5. `bpf_text`：定义了一个eBPF程序的源代码。该程序会在每次调用`readline`函数时，将返回的字符串和进程ID发送到一个`perf`事件中。

6. `STR_DATA = 80`：定义了一个常量，表示字符串的最大长度。

7. `class Data(ct.Structure)`：定义了一个`ctypes`结构体，用于表示事件数据。该结构体包含一个进程ID和一个长度为80的字符串。

8. `b = BPF(text=bpf_text)`：创建一个`BPF`对象，并将eBPF程序源代码传递给它进行编译和加载。

9. `b.attach_uretprobe(name="/bin/bash", sym="readline", fn_name="printret")`：将eBPF程序附加到`readline`函数的返回值上，以便在每次调用`readline`函数时触发程序。

10. `print("%-9s %-6s %s" % ("TIME", "PID", "COMMAND"))`：打印表头，用于显示输出的格式。

11. `def print_event(cpu, data, size)`：定义一个回调函数，用于处理`perf`事件。该函数会将事件数据转换为`Data`结构体，并打印出时间戳、进程ID和字符串。

12. `b["events"].open_perf_buffer(print_event)`：打开一个`perf`事件缓冲区，用于接收事件数据并调用`print_event`函数进行处理。

13. `while 1: b.kprobe_poll()`：进入一个无限循环，不断调用`kprobe_poll`函数来处理事件缓冲区中的数据。

总的来说，这段代码使用`bcc`库加载了一个eBPF程序，该程序会在每次调用`readline`函数时触发，并将返回的字符串和进程ID发送到一个`perf`事件中。然后，通过回调函数将事件数据打印出来。
```

### eBPF故障排查

#### 内存限制

`eBPF map`使用固定的内存（`locked memory`），但默认非常小，可以通过调用[setrlimit(2)](http://man7.org/linux/man-pages/man2/setrlimit.2.html)来增大`RLIMIT_MEMLOCK`。如果内存不足，`bpf_create_map`会返回`EPERM (Operation not permitted)`错误。

#### 开启BPF JIT

```sh
$ sysctl net/core/bpf_jit_enable=1
net.core.bpf_jit_enable = 1
```

```
开启BPF JIT（Just-In-Time）是为了提高eBPF（Extended Berkeley Packet Filter）的性能。eBPF是一种虚拟机技术，用于在Linux内核中执行高性能的网络数据包过滤和处理。BPF JIT允许将eBPF程序编译成本地机器码，以便在运行时进行即时编译和执行，从而提高执行效率。

通过开启BPF JIT，可以将eBPF程序转换为本地机器码，避免了解释执行的性能损耗。这样可以显著提高eBPF程序的执行速度，使其能够更高效地处理网络数据包，实现更复杂的网络过滤和处理功能。

开启BPF JIT还可以提供更高的灵活性和扩展性，因为它允许在运行时动态编译和执行eBPF程序。这意味着可以根据实际需求动态修改和优化eBPF程序，以适应不同的网络环境和需求。

总之，开启BPF JIT可以提高eBPF的性能和灵活性，使其成为一种强大的工具，用于高效地处理和过滤网络数据包。
```

#### ELF二进制文件

`eBPF`通过`LLVM`编译器生成的程序就是一个普通的`ELF`二进制文件，可以使用`readelf`或者`llvm-objdump`分析该文件，如

```sh
$ llvm-objdump -h xdp_ddos01_blacklist_kern.o

xdp_ddos01_blacklist_kern.o:    file format ELF64-unknown

Sections:
Idx Name          Size      Address          Type
  0               00000000 0000000000000000
  1 .strtab       00000072 0000000000000000
  2 .text         00000000 0000000000000000 TEXT DATA
  3 xdp_prog      000001b8 0000000000000000 TEXT DATA
  4 .relxdp_prog  00000020 0000000000000000
  5 maps          00000028 0000000000000000 DATA
  6 license       00000004 0000000000000000 DATA
  7 .symtab       000000d8 0000000000000000
```

#### 提取eBPF-JIT代码

在调试`eBPF`程序时，有时需要提取`eBPF-JIT`代码

```sh
$ sysctl net.core.bpf_jit_enable=2
```

输出如下所示：

```
 flen=55 proglen=335 pass=4 image=ffffffffa0006820 from=xdp_ddos01_blac pid=13333
 JIT code: 00000000: 55 48 89 e5 48 81 ec 28 02 00 00 48 89 9d d8 fd
 JIT code: 00000010: ff ff 4c 89 ad e0 fd ff ff 4c 89 b5 e8 fd ff ff
 JIT code: 00000020: 4c 89 bd f0 fd ff ff 31 c0 48 89 85 f8 fd ff ff
 JIT code: 00000030: bb 02 00 00 00 48 8b 77 08 48 8b 7f 00 48 89 fa
 JIT code: 00000040: 48 83 c2 0e 48 39 f2 0f 87 e1 00 00 00 48 0f b6
 JIT code: 00000050: 4f 0c 48 0f b6 57 0d 48 c1 e2 08 48 09 ca 48 89
 JIT code: 00000060: d1 48 81 e1 ff 00 00 00 41 b8 06 00 00 00 49 39
 JIT code: 00000070: c8 0f 87 b7 00 00 00 48 81 fa 88 a8 00 00 74 0e
 JIT code: 00000080: b9 0e 00 00 00 48 81 fa 81 00 00 00 75 1a 48 89
 JIT code: 00000090: fa 48 83 c2 12 48 39 f2 0f 87 90 00 00 00 b9 12
 JIT code: 000000a0: 00 00 00 48 0f b7 57 10 bb 02 00 00 00 48 81 e2
 JIT code: 000000b0: ff ff 00 00 48 83 fa 08 75 49 48 01 cf 31 db 48
 JIT code: 000000c0: 89 fa 48 83 c2 14 48 39 f2 77 38 8b 7f 0c 89 7d
 JIT code: 000000d0: fc 48 89 ee 48 83 c6 fc 48 bf 00 9c 24 5f 07 88
 JIT code: 000000e0: ff ff e8 29 cd 13 e1 bb 02 00 00 00 48 83 f8 00
 JIT code: 000000f0: 74 11 48 8b 78 00 48 83 c7 01 48 89 78 00 bb 01
 JIT code: 00000100: 00 00 00 89 5d f8 48 89 ee 48 83 c6 f8 48 bf c0
 JIT code: 00000110: 76 12 13 04 88 ff ff e8 f4 cc 13 e1 48 83 f8 00
 JIT code: 00000120: 74 0c 48 8b 78 00 48 83 c7 01 48 89 78 00 48 89
 JIT code: 00000130: d8 48 8b 9d d8 fd ff ff 4c 8b ad e0 fd ff ff 4c
 JIT code: 00000140: 8b b5 e8 fd ff ff 4c 8b bd f0 fd ff ff c9 c3
```

其中，`proglen`是`opcode sequence`的长度，`flen`是`bpf insns`的个数。可以使用`bpf_jit_disasm`工具来生成相关的`opcodes`。

## XDP

XDP（eXpress Data Path）是一种高性能网络数据包处理技术，它在Linux内核中实现。XDP可以在网络接口的数据包处理路径上进行操作，以实现快速的数据包转发和处理。

XDP的出现主要是为了应对高速网络环境下的数据包处理需求。传统的网络数据包处理方式通常涉及多次内核态和用户态之间的切换，这会引入较大的延迟和性能损失。而XDP通过在内核中实现一套基于eBPF（extended Berkeley Packet Filter）的虚拟机来处理数据包，避免了不必要的上下文切换，提供了更高的性能和更低的延迟。

XDP的主要应用场景包括：

1. 数据包过滤和转发：XDP可以实现高效的数据包过滤和转发，可以根据数据包的内容进行筛选和重定向，以满足不同的网络需求。
2. DDoS防护：XDP可以在网络接口上实现快速的DDoS攻击检测和防护，能够在数据包到达内核之前进行快速处理，有效减轻服务器负载。
3. 网络监控和分析：XDP可以在数据包处理路径上进行灵活的监控和分析，可以实时捕获和处理网络数据包，提供更高效的网络监控和故障排查能力。

总之，XDP通过在内核中实现高性能的数据包处理虚拟机，提供了更低延迟和更高吞吐量的网络数据包处理能力，满足了高速网络环境下对性能和效率的需求。

XDP为Linux内核提供了高性能、可编程的网络数据路径。由于网络包在还未进入网络协议栈之前就处理，它给Linux网络带来了巨大的性能提升（性能比DPDK还要高）。

![xdp-packet-processing](https://tonydeng.gitbooks.io/sdn/content/linux/XDP/images/xdp-packet-processing-1024x560.png)

XDP主要的特性包括

* 在网络协议栈前处理
* 无锁设计
* 批量I/O操作
* 轮询式
* 直接队列访问
* 不需要分配skbuff
* 支持网络卸载
* DDIO
* XDP程序快速执行并结束，没有循环
* Packeting steering

### XDP架构

XDP基于一系列的技术来实现高性能和可编程性，包括

* 基于eBPF
* Capabilities negotiation：通过协商确定网卡驱动支持的特性，XDP尽量利用新特性，但网卡驱动不需要支持所有的特性
* 在网络协议栈前处理
* 无锁设计
* 批量I/O操作
* 轮询式
* 直接队列访问
* 不需要分配skbuff
* 支持网络卸载
* DDIO
* XDP程序快速执行并结束，没有循环
* Packeting steering

包处理逻辑：如下图所示，基于内核的eBPF程序处理包，每个RX队列分配一个CPU，且以每个网络包一个Page（packet-page）的方式避免分配skbuff。

![xdp-packet-processor](https://tonydeng.gitbooks.io/sdn/content/linux/XDP/images/packet-processor.png)

### 与DPDK对比

相对于DPDK，XDP具有以下优点

* 无需第三方代码库和许可
* 同时支持轮询式和中断式网络
* 无需分配大页
* 无需专用的CPU
* 无需定义新的安全网络模型

### 示例

* [Linux内核BPF示例](https://github.com/torvalds/linux/tree/master/samples/bpf)
* [prototype-kernel示例](https://github.com/netoptimizer/prototype-kernel/tree/master/kernel/samples/bpf)
* [libbpf](https://github.com/torvalds/linux/tree/master/tools/lib/bpf)

### 缺点

注意XDP的性能提升是有代价的，它牺牲了通用型和公平性

* XDP不提供缓存队列（qdisc），TX设备太慢时直接丢包，因而不要在RX比TX快的设备上使用XDP
* XDP程序是专用的，不具备网络协议栈的通用性

### XDP使用场景

XDP的使用场景包括

* DDoS防御
* 防火墙
* 基于`XDP_TX`的负载均衡
* 网络统计
* 复杂网络采样
* 高速交易平台

### 参考

* [Introduction to XDP](https://www.iovisor.org/technology/xdp)
* [Network Performance BoF](http://people.netfilter.org/hawk/presentations/NetDev1.1\_2016/links.html)
* [XDP Introduction and Use-cases](http://people.netfilter.org/hawk/presentations/xdp2016/xdp\_intro\_and\_use\_cases\_sep2016.pdf)
* [Linux Network Stack](http://people.netfilter.org/hawk/presentations/theCamp2016/theCamp2016\_next\_steps\_for\_linux.pdf)
* [NetDev 1.2 video](https://www.youtube.com/watch?v=NlMQ0i09HMU\&feature=youtu.be\&t=3m3s)
* [XDP Hands-On Tutorial](https://github.com/xdp-project/xdp-tutorial)

## 常用工具

### tcpdump

参考：[细说tcpdump的妙用](https://community.emc.com/message/854940#854940)

tcpdump命令：

```sh
tcpdump -en -i p3p2 -vv     # show vlan

tcpdump是一个网络抓包工具，用于捕获和分析网络数据包。下面是对给出的命令的解释：

- tcpdump：运行tcpdump命令。
- -en：以以太网帧格式显示捕获的数据包。
- -i p3p2：指定要监听的网络接口为p3p2。
- -vv：增加详细的输出信息，包括更多的协议头信息和解析。
```

tcpdump选项可划分为四大类型：控制tcpdump程序行为，控制数据怎样显示，控制显示什么数据，以及过滤命令。

#### 控制程序行为

这一类命令行选项影响程序行为，包括数据收集的方式。之前已介绍了两个例子：-r和-w。-w选项允许用户将输出重定向到一个文件，之后可通过-r选项将捕获数据显示出来。

如果用户知道需要捕获的报文数量或对于数量有一个上限，可使用-c选项。则当达到该数量时程序自动终止，而无需使用kill命令或Ctrl-C。下例中，收集到100个报文之后tcpdump终止：

```sh
bsd1# tcpdump -c100
```

如果用户在多余一个网络接口上运行tcpdump，用户可以通过-i选项指定接口。在不确定的情况下，可使用ifconfig –a来检查哪一个接口可用及对应哪一个网络。例如，一台机器有两个C级接口，xl0接口IP地址205.153.63.238，xl1接口IP地址205.153.61.178。要捕捉205.153.61.0网络的数据流，使用以下命令：

```sh
bsd1# tcpdump -i xl1
```

没有指定接口时，tcpdump默认为最低编号接口。

`-p`选项将网卡接口设置为非混杂模式。这一选项理论上将限制为捕获接口上的正常数据流——来自或发往主机，多播数据，以及广播数据。

`-s`选项控制数据的截取长度。通常，tcpdump默认为一最大字节数量并只会从单一报文中截取到该数量长度。实际字节数取决于操作系统的设备驱动。通过默认值来截取合适的报文头，而舍弃不必要的报文数据。

如果用户需截取更多数据，通过-s选项来指定字节数。也可以用-s来减少截取字节数。对于少于或等于200字节的报文，以下命令会截取完整报文：

```sh
bsd1# tcpdump -s200
```

更长的报文会被缩短为200字节。

#### 控制信息如何显示

`-a`，`-n`，`-N`和`-f`选项决定了地址信息是如何显示的。-a选项强制将网络地址显示为名称，-n阻止将地址显示为名字，-N阻止将域名转换。-f选项阻止远端名称解析。下例中，从sloan.lander.edu (205.153.63.30) ing远程站点，分别不加选项，-a，-n，-N，-f。（选项-c1限制抓取1个报文）

```bash
bsd1# tcpdump -c1 host 192.31.7.130
tcpdump: listening on xl0
14:16:35.897342 sloan.lander.edu > cio-sys.cisco.com: icmp: echo request
bsd1# tcpdump -c1 -a host 192.31.7.130
tcpdump: listening on xl0
14:16:14.567917 sloan.lander.edu > cio-sys.cisco.com: icmp: echo request
bsd1# tcpdump -c1 -n host 192.31.7.130
tcpdump: listening on xl0
14:17:09.737597 205.153.63.30 > 192.31.7.130: icmp: echo request
bsd1# tcpdump -c1 -N host 192.31.7.130
tcpdump: listening on xl0
14:17:28.891045 sloan > cio-sys: icmp: echo request
bsd1# tcpdump -c1 -f host 192.31.7.130
tcpdump: listening on xl0
14:17:49.274907 sloan.lander.edu > 192.31.7.130: icmp: echo request
```

默认为-a选项。

`-t`和`-tt`选项控制时间戳的打印。-t选项不显示时间戳而-tt选项显示无格式的时间戳。以下命令显示了tcpdump命令无选项，-t选项，-tt选项的同一报文：

```sh
12:36:54.772066 sloan.lander.edu.1174 > 205.153.63.238.telnet: . ack 3259091394 win 8647 (DF)
sloan.lander.edu.1174 > 205.153.63.238.telnet: . ack 3259091394 win 8647 (DF)
934303014.772066 sloan.lander.edu.1174 > 205.153.63.238.telnet: . ack 3259091394 win 8647 (DF)
```

#### 控制显示什么数据

可以通过-v和-vv选项来打印更多详细信息。例如，-v选项将会打印TTL字段。要显示较少信息，使用-q，或quiet选项。一下为同一报文分别使用-q选项，无选项，-v选项，和-vv选项的输出。

```sh
12:36:54.772066 sloan.lander.edu.1174 > 205.153.63.238.telnet: tcp 0 (DF)
12:36:54.772066 sloan.lander.edu.1174 > 205.153.63.238.telnet: . ack 3259091394 win 8647 (DF)
12:36:54.772066 sloan.lander.edu.1174 > 205.153.63.238.telnet: . ack 3259091394 win 8647 (DF) (ttl 128, id 45836)
12:36:54.772066 sloan.lander.edu.1174 > 205.153.63.238.telnet: . ack 3259091394 win 8647 (DF) (ttl 128, id 45836)
```

`-e`选项用于显示链路层头信息。上例中-e选项的输出为：

```sh
12:36:54.772066 0:10:5a:a1:e9:8 0:10:5a:e3:37:c ip 60:
sloan.lander.edu.1174 > 205.153.63.238.telnet: . ack 3259091394 win 8647 (DF)
```

0:10:5a:a1:e9:8是sloan.lander.edu中3Com卡的以太网地址，0:10:5a:e3:37:c是205.153.63.238中3Com卡的以太网地址。

`-x`选项将报文以十六进制形式dump出来，排除了链路层报文头。-x和-vv选项报文显示如下：

```sh
13:57:12.719718 bsd1.lander.edu.1657 > 205.153.60.5.domain: 11587+ A? www.microsoft.com. (35) (ttl 64, id 41353)
                         4500 003f a189 0000 4011 c43a cd99 3db2
                         cd99 3c05 0679 0035 002b 06d9 2d43 0100
                         0001 0000 0000 0000 0377 7777 096d 6963
                         726f 736f 6674 0363 6f6d 0000 0100 01
```

#### 过滤

要有效地使用tcpdump，掌握过滤器非常必要的。过滤允许用户指定想要抓取的数据流，从而用户可以专注于感兴趣的数据。此外，ethereal这样的工具使用tcpdump过滤语法来抓取数据流。

如果用户很清楚对何种数据流不感兴趣，可以将这部分数据排除在外。如果用户不确定需要什么数据，可以将源数据收集到文件之后在读取时应用过滤器。实际应用中，需要经常在两种方式之间转换。

简单的过滤器是加在命令行之后的关键字。但是，复杂的命令是由逻辑和关系运算符构成的。对于这样的情况，通常最好用-F选项将过滤器存储在文件中。例如，假设testfilter 是一个包含过滤主机205.153.63.30的文本文件，之后输入tcpdump –Ftestfilter等效于输入命令tcpdump host 205.153.63.30。通常，这一功能只在复杂过滤器时使用。但是，同一命令中命令行过滤器和文件过滤器不能混用。

**地址过滤**

过滤器可以按照地址选择数据流。例如，考虑如下命令：

```sh
bsd1# tcpdump host 205.153.63.30
```

该命令抓取所有来自以及发往IP地址205.153.63.30的主机。主机可以通过名称或IP地址来选定。虽然指定的是IP地址，但抓取数据流并不限于IP数据流，实际上，过滤器也会抓到ARP数据流。限定仅抓取特定协议的数据流要求更复杂的过滤器。

有若干种方式可以指定和限制地址，下例是通过机器的以太网地址来选择数据流：

```sh
bsd1# tcpdump ether host 0:10:5a:e3:37:c
```

数据流可进一步限制为单向，分别用src或dst指定数据流的来源或目的地。下例显示了发送到主机205.153.63.30的数据流：

```sh
bsd1# tcpdump dst 205.153.63.30
```

注意到本例中host被省略了。在某些例子中省略是没问题的，但添加这些关键字通常更安全些。

广播和多播数据相应可以使用broadcast和multicast。由于多播和广播数据流在链路层和网络层所指定的数据流是不同的，所以这两种过滤器各有两种形式。过滤器ether multicast抓取以太网多播地址的数据流，ip multicast抓取IP多播地址数据流。广播数据流也是类似的使用方法。注意多播过滤器也会抓到广播数据流。

除了抓取特定主机以外，还可以抓取特定网络。例如，以下命令限制抓取来自或发往205.153.60.0的报文：

```sh
bsd1# tcpdump net 205.153.60
```

以下命令也可以做同样的事情：

```sh
bsd1# tcpdump net 205.153.60.0 mask 255.255.255.0
```

而以下命令由于最后的.0就无法正常工作：

```sh
bsd1# tcpdump net 205.153.60.0
```

```sh
这是因为在tcpdump命令中，如果使用CIDR（无类域间路由）表示法指定网络，最后的.0会被视为无效的。CIDR表示法是将IP地址和掩码一起表示为一个单独的值，例如205.153.60.0/24表示网络地址为205.153.60.0，子网掩码为255.255.255.0。

因此，如果要限制抓取来自或发往205.153.60.0的报文，可以使用以下命令：

bsd1# tcpdump net 205.153.60.0/24
```

**协议及端口过滤**

限制抓取指定协议如IP，Appletalk或TCP。还可以限制建立在这些协议之上的服务，如DNS或RIP。这类抓取可以通过三种方式进行：使用tcpdump关键字，通过协议关键字proto，或通过服务使用port关键字。

一些协议名能够被tcpdump识别到因此可通过关键字来指定。以下命令限制抓取IP数据流：

```sh
bsd1# tcpdump ip
```

当然，IP数据流包括TCP数据流，UDP数据流，等等

如果仅抓取TCP数据流，可以使用：

```sh
bsd1# tcpdump tcp
```

tcpdump可识别的关键字包括ip, igmp, tcp, udp, and icmp。

有很多传输层服务没有可以识别的关键字。在这种情况下，可以使用关键字proto或ip proto加上/etc/protocols能够找到的协议名或相应的协议编号。例如，以下两种方式都会查找OSPF报文：

```sh
bsd1# tcpdump ip proto ospf
bsd1# tcpdump ip proto 89
```

内嵌的关键字可能会造成问题。下面的例子中，无法使用tcp关键字，或必须使用数字。例如，下面的例子是正常工作的：

```sh
bsd#1 tcpdump ip proto 6
```

另一方面，不能使用proto加上tcp:

```sh
bsd#1 tcpdump ip proto tcp
```

会产生问题。

对于更高层级的建立于底层协议之上的服务，必须使用关键字port。以下两者会采集DNS数据流：

```sh
bsd#1 tcpdump port domain
bds#1 tcpdump port 53
```

第一条命令中，关键字domain能够通过查找/etc/services来解析。在传输层协议有歧义的情况下，可以将端口限制为指定协议。考虑如下命令：

```sh
bsd#1 tcpdump udp port domain
```

这会抓取使用UDP的DNS名查找但不包括使用TCP的DNS zone传输数据。而之前的两条命令会同时抓取这两种数据。

**报文特征**

过滤器也可以基于报文特征比如报文长度或特定字段的内容，过滤器必须包含关系运算符。要指定长度，使用关键字less或greater。如下例所示：

```sh
bsd1# tcpdump greater 200
```

该命令收集长度大于200字节的报文。

根据报文内容过滤更加复杂，因为用户必须理解报文头的结构。但是尽管如此，或者说正因如此，这一方式能够使用户最大限度的控制抓取的数据。

一般使用语法 proto \[ expr : size ]。字段proto指定要查看的报文头——ip则查看IP头，tcp则查看TCP头，以此类推。expr字段给出从报文头索引0开始的位移。即：报文头的第一个字节为0，第二字节为1，以此类推。size字段是可选的，指定需要使用的字节数，1，2或4。

```sh
bsd1# tcpdump "ip[9] = 6"
```

查看第十字节的IP头，协议值为6。注意这里必须使用引号。撇号或引号都可以，但反引号将无法正常工作。

```sh
bsd1# tcpdump tcp
```

也是等效的，因为TCP协议编号为6。

这一方式常常作为掩码来选择特定比特位。值可以是十六进制。可通过语法&加上比特掩码来指定。下例提取从以太网头第一字节开始（即目的地址第一字节），提取低阶比特位，并确保该位不为0：

```sh
bsd1# tcpdump 'ether[0] & 1 != 0'
```

该条件会选取广播和多播报文。

以上两个例子都有更好的方法来匹配报文。作为一个更实际的例子，考虑以下命令：

```sh
bsd1# tcpdump "tcp[13] & 0x03 != 0"
```

该过滤器跳过TCP头的13个字节，提取flag字节。掩码0x03选择第一和第二比特位，即FIN和SYN位。如果其中一位不为0则报文被抓取。此命令会抓取TCP连接建立及关闭报文。

不要将逻辑运算符与关系运算符混淆。比如想`tcp src port > 23`这样的表达式就无法正常工作。因为tcp src port表达式返回值为true或false，而不是一个数值，所以无法与数值进行比较。如果需要查找端口号大于23的所有TCP数据流，必须从报文头提取端口字段，使用表达式`tcp[0:2] & 0xffff > 0x0017`。

### scapy

`scapy`是一个强大的`python`网络数据包处理库，它可以生成或解码网络协议数据包，可以用来端口扫描、探测、网络测试等。

#### scapy安装

```bash
pip install scapy
```

#### 简单使用

`scapy`提供了一个简单的交互式界面，直接运行`scapy`命令即可进入。当然，也可以在`python`交互式命令行中导入`scapy`包进入

```python
from scapy.all import *
```

查看所有支持的协议和预制工具：

```python
ls()
lsc()
```

构造IP数据包:

```python
pkt=IP(dst="8.8.8.8")
pkt.show()
print pkt.dst  # 8.8.8.8
str(pkt)       # hex string
hexdump(pkt)   # hex dump
```

输出HEX格式的数据包：

`scapy`提供了一个简单的交互式界面，直接运行`scapy`命令即可进入。当然，也可以在`python`交互式命令行中导入`scapy`包进入

```python
from scapy.all import *
```

查看所有支持的协议和预制工具：

```python
ls()
lsc()
```

构造IP数据包

```python
pkt=IP(dst="8.8.8.8")
pkt.show()
print pkt.dst  # 8.8.8.8
str(pkt)       # hex string
hexdump(pkt)   # hex dump
```

输出HEX格式的数据包

```py
import binascii
from scapy.all import *
a=Ether(dst="02:ac:10:ff:00:22",src="02:ac:10:ff:00:11")/IP(dst="172.16.255.22",src="172.16.255.11", ttl=10)/ICMP()
print binascii.hexlify(str(a))
```

TCP/IP协议的四层模型都可以分别构造，并通过/连接：

```py
Ether()/IP()/TCP()
IP()/TCP()
IP()/TCP()/"GET / HTTP/1.0\r\n\r\n"
Ether()/IP()/IP()/UDP()
Ether()/IP(dst="www.slashdot.org")/TCP()/"GET /index.html HTTP/1.0 \n\n"
```

从PCAP文件读入数据：

```py
a = rdpcap("test.cap")
```

发送数据包：

```py
# 三层发送，不接收
send(IP(dst="8.8.8.8")/ICMP())  
# 二层发送，不接收
sendp(Ether()/IP(dst="8.8.8.8",ttl=10)/ICMP())
# 三层发送并接收
# 二层可以用srp, srp1和srploop
result, unanswered = sr(IP(dst="8.8.8.8")/ICMP())
# 发送并只接收第一个包
sr1(IP(dst="8.8.8.8")/ICMP())
# 发送多个数据包
result=srloop(IP(dst="8.8.8.8")/ICMP(), inter=1, count=2)
```

嗅探数据包：

```py
sniff(filter="icmp", count=3, timeout=5, prn=lambda x:x.summary())
a=sniff(filter="tcp and ( port 25 or port 110 )",
 prn=lambda x: x.sprintf("%IP.src%:%TCP.sport% -> %IP.dst%:%TCP.dport%  %2s,TCP.flags% : %TCP.payload%"))
```

SYN扫描：

```py
sr1(IP(dst="172.217.24.14")/TCP(dport=80,flags="S"))
ans,unans = sr(IP(dst=["192.168.1.1","yahoo.com","slashdot.org"])/TCP(dport=[22,80,443],flags="S"))
```

TCP traceroute：

```py
ans,unans=sr(IP(dst="www.baidu.com",ttl=(2,25),id=RandShort())/TCP(flags=0x2),timeout=50)
for snd,rcv in ans:
    print snd.ttl,rcv.src,isinstance(rcv.payload,TCP)
```

ARP Ping：

```py
ans,unans=srp(Ether(dst="ff:ff:ff:ff:ff:ff")/ARP(pdst="192.168.1.0/24"),timeout=2)
ans.summary(lambda (s,r): r.sprintf("%Ether.src% %ARP.psrc%") )
```

ICMP Ping:

```py
ans,unans=sr(IP(dst="192.168.1.1-254")/ICMP())
ans.summary(lambda (s,r): r.sprintf("%IP.src% is alive") )
```

TCP Ping:

```py
ans,unans=sr( IP(dst="192.168.1.*")/TCP(dport=80,flags="S") )
ans.summary( lambda(s,r) : r.sprintf("%IP.src% is alive") )
```
