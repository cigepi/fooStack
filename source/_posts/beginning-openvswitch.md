title: 对 OpenvSwitch 的业余理解
date: 2013-07-11 20:46
categories: Tech Logs
tags:
- OpenStack
- OpenvSwitch
- vlan
---

本人计算机网络基础很差，现在为了兴趣所在的 OpenStack，要试图理解 OpenvSwitch。对于要学习在不熟悉的领域的非常复杂东西的人来说，兴趣与方法很重要。推荐这篇翻译了费曼技巧的[博文](http://way4ever.com/?p=377)

本文仅以**非常业余**的角度，通过简述、转述 [OpenvSwitch FAQ](http://git.openvswitch.org/cgi-bin/gitweb.cgi?p=openvswitch;a=blob_plain;f=FAQ;hb=HEAD)，来试图帮助自己与读者理解 OpenvSwitch。

本文假设在读者已了解什么是交换机的端口链路类型，什么是 Linux 网桥（Linux Bridge）的基础上进行阐述。

**Q: 什么是 OpenvSwitch？**

**A: **OpenvSwitch 是一个生产环境质量级的、开源的虚拟交换机软件。在虚拟化环境中，单个物理机的上的不同虚拟机、多个物理机上的不同虚拟机之间通过它进行网络通信。OpenvSwitch 被设计为可以与当下流行的交换机芯片兼容，这意味着它可以与物理交换机协同工作，并提供自由、灵活的管理手段。

---

**Q: OpenvSwtich 支持哪些虚拟化平台？**

**A: **OpenvSwitch 支持运行在 Linux上 的下列虚拟化平台: KVM, VirtualBox, Xen, Xen Cloud Platform, XenServer.

---

**Q: 怎样获取 OpenvSwitch?**

**A: **可以在 Linux 下编译源码；Debian, Ubuntu, Fedora的软件仓库中已有软件包，可以直接安装使用；下载最新的 XenServer 或 Xen Cloud Platform 的 ISO 镜像，它们已经整合了 OpenvSwitch。

---

**Q: 为何要用 OpenvSwitch 替换 Linux Bridge?**

**A: **相比于简单的 Linux Bridge，OpenvSwitch 可以实现网络隔离、QoS 配置、流量监控、数据包分析等物理交换网络所具有的功能。

---

**Q: OpenvSwitch 与 VMware vNetwork distributed switch or the Cisco Nexus 1000V 这样的分布式虚拟交换应用有何不同？**

**A: **分布式虚拟交换应用提供一个集中管理、监控远程主机上的网络的功能。而 OpenvSwitch 本身不具备分布式架构，OpenvSwitch 单独运行在每个物理主机上，而由上层管理软件来提供分布式特性。

---

**Q: 为何 OpenvSwitch 不是分布式架构？**

**A: **OpenvSwitch 意图成为一个被用来自由构建网络的实用组件。分布式网络系统需要平衡简单性、可扩展性、硬件兼容性、收敛时间等。相比于实现一套有特定优劣的系统，OpenvSwitch 选择成为一个基本组件。

---

**Q: 我把 OpenvSwitch 认作是一个虚拟以太网交换机，但是为何文档中不断的提到 `bridge`，什么是 `bridge`？**

**A: **在计算机网络中，术语 `bridge` 与 `switch` 是同义词。OpenvSwitch 实现一个以太网交换机（Ethernet switch），与实现一个以太网桥（Ethernet Bridge）是几个意思，奥不对，是一个意思。

---

**Q: 如何将一个端口配置为 ACCESS 端口？**

**A: **将 `tag=VLAN`，添加至你的 `ovs-vsctl add-port` 命令中。

例如，下列命令将 eth0 配置为 br0 的 trunk 端口。tap0 配置为 br0 的 access 端口，属于 VLAN9。

```
ovs-vsctl add-br br0
ovs-vsctl add-port br0 eth0
ovs-vsctl add-port br0 tap0 tag=9
```

如果想将一个已经添加了的端口设置为 access 端口，可以使用 `ovs-vsctl set` 命令，例如：

```
ovs-vsctl set port tap0 tag=9
```

---

**Q: 我创建了一个 bridge，然后把我的物理网卡与其绑定了，就像这样：**

```
ovs-vsctl add-br br0
ovs-vsctl add-port br0 eth0
```

然后，然后我的 eth0 网卡就宕了！救命！(=_=)

**A: **一块物理以太网卡如果作为 OpenvSwitch bridge 的一部分，则它不能拥有 IP 地址，如果有，也会完全不起作用。如果发生了上述情况，你可以将 IP 地址绑定至某 OpenvSwitch "internal" 设备来恢复网络访问功能。

例如，假设你的 eth0 IP 地址为 192.168.128.5，在执行上文问题中的命令后，你可以使用如下方法来将 IP 地址绑定至br0 上：

```
ifconfig eth0 0.0.0.0
ifconfig br0 192.168.128.5
```

---

**Q: 我创建了一个 bridge，然后我把两块物理网卡绑定到了上面，就像这样：**

```
ovs-vsctl add-br br0
ovs-vsctl add-port br0 eth0
ovs-vsctl add-port br0 eth1
```

然后，我的网络访问就完全混乱了！CPU 利用率也非常高！(=_=)

**A: **基本上，你把你的网络打成一个环了。

在上面的设置下，OpenvSwitch 在 eth0 上收到一个广播包后会将其发给 eth1，然后 eth1 上的物理交换机又将这个广播包发还给 eth0，如此往复。当有多个 switch 时，还会产生更复杂的情况。

解决方案1：

如果你想将 eth0 与 eth1 都绑定至同一个 bridge 从而获得更大的带宽、更高的可靠性，可以像下面这样做：

```
ovs-vsctl add-br br0
ovs-vsctl add-bond br0 bond0 eth0 eth1
```

解决方案2：

如果你不想把他们放到一块，你可以弄两个 bridge 嘛：

```
ovs-vsctl add-br br0
ovs-vsctl add-port br0 eth0
ovs-vsctl add-br br1
ovs-vsctl add-port br1 eth1
```

解决方案3：

如果你已经拥有了一个复杂或冗余的网络拓扑结构，但你想预防结环，你就需要打开生成树协议(spanning tree protocol, STP).

按照下面的命令序列，依序创建 br0, 打开 STP, 然后将 eth0 与 eth1 绑定至 br0.

```
ovs-vsctl add-br br0
ovs-vsctl set bridge br0 stp_enable=true
ovs-vsctl add-port br0 eth0
ovs-vsctl add-port br0 eth1
```

---

**Q: 我好像不能在无线网络中使用 OpenvSwitch？**

**A: **不可以，Linux Bridge 也不可以。

---

**Q: 有关于 OpenvSwitch 数据库的表结构的文档么？**

**A: **有的，`ovs-vswitchd.conf.db(5)` 有详细解释。

---

**Q: 我运行 `ovs-dpctl` 的时候看不到已创建的 bridge，而只看到一个名叫 `ovs-system` 的 `datapath`。我怎样能看到特定 bridge 的 datapath？**

**A: **在 OpenvSwitch 1.9.0 版中，所有 switch 都共享单个 datapath。命令 `ovs-appctl dpif/*` 提供与 ovs-dpctl 类似的 datapath 信息，但是按照不同的 bridge 区分。

---

**Q: 什么是VLAN？**

**A: **最简单的来说，VLAN（Virtual LAN 的缩写）技术就是把一个交换机划分为多个交换机的技术。举个栗子，你有两组机器 A 与 B，你想让 A 与 B 中的机器只能组内访问，两个交换机当然可以，但是如果你只有一个交换机，你就可以使用 VLAN技术来做同样的事。通过设置 A 中机器为 VLAN1 的 `access port`，B 中的机器为 VLAN2 的 `access port`，使交换机仅仅会在同一 VLAN 的端口之间转发数据包。这样你就把一个交换机划分为两个独立的交换机来使用了。
