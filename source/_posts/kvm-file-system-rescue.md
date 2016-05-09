title: OpenStack 之 KVM 虚拟机文件系统修复
date: 2013-02-27 15:08
categories: Tech Logs
tags:
- filesystem
- KVM
- OpenStack
---

本文描述了在 OpenStack 环境下的 KVM 虚拟机文件系统损坏后，如何修复文件系统并找回数据。

## 故障现象

在一次物理机非法关机后，任何对虚拟机文件系统的写入操作均提示：Read-only file system，即文件系统不可写。

虚拟机分区表：

```shell-session
/dev/vda1	挂载于	/boot	#文件系统正常
/dev/vda2	未挂载
/dev/vda3	挂载于	/		#文件系统 read-only
/dev/vdb1	挂载于	/home	#文件系统 raed-only
```

**注：如果把此虚拟机理解为一台 PC 的话，`vda` 相当于自带磁盘，`vdb` 相当于一块移动硬盘。**

## 处理方案

- **`A1`** 使用 OpenStack 建立该虚拟机（以下简称 `old_instance`）的快照（Snapshot），将该 snapshot 从其他物理机（主机，Host）启动，以恢复虚拟机 vda。

- **`A2`** 对于 vdb，创建快照并不能保存其数据，vdb 对于 old_instance 来说，就相当于一块移动硬盘。

- **`B1`** 首先将 vdb1（`/home`分区）内的数据使用 `scp` 备份到异地保存，再将 vdb1 卸载，使用 `fsck` 修复磁盘，即可恢复 vdb1。

- **`B2`** 使用该方案，无法修复 vda1（`/boot`分区）与 vda3（`/`分区），因为无法将该两个分区卸载，在线使用 `fsck` 可能造成数据损坏。

考虑全面后，将两种方式结合，即**`A1+B1`**，即可恢复该虚拟机。


## A1+B1操作过程

1.	为 instance 建立快照，使用命令

	```console
	# nova image-create old_instance_name snp_name
	```

2.	使用该快照，在另一台 host 启动 new_instance，使用命令

	```console
	# nova boot --image snp_name --flavor n --availability_zone xxx:host_name new_instance_name
	```

3.	启动该 new_instance 后，使用 VNC 客户端连接至该 instance，服务器因报错停留在此处：

	![启动报错](http://hiaero.net/wp-content/uploads/2013/02/kvm_rescue_pic01-1024x631.jpg)

	使用 snapshot 启动后，instance 启动报错停留在此处

4.	在上图处，输入 root 密码，登入一个维护命令行，此时该 instance 的各个分区已挂载好了，但是分区仍然处于 read-only 模式。
5.	此时依次启动网卡与 ssh 服务，将除 `/home` 分区外的文件 `scp` 至异地保存，再使用 `fsck` 命令对 vda1 与 vda3 进行修复，`fsck` 命令执行完毕后，输入 `reboot` 重启系统，new_instance 正常启动。
6.	在 old_instance 将 `/home` 分区下的文件传送到 new_instance，迁移完成，此方式可以将故障机快速恢复。


## 延伸操作: KVM 虚拟机进入救援模式恢复文件系统

一般情况下服务器文件系统损坏后，可以使用 ilo/idrac 挂载 iso 镜像进入 linux rescue mode 对文件系统进行修复。KVM 虚拟机也可以使用 [libvrit](http://www.libvirt.org/) 编辑其 XML 文件来达到这个目的，下面使用该方法来对 old_instance 进行恢复。

**注：old_instance_domain 代表 old_instance 的 domain name，可以通过 `nova show old_instance_name` 命令查看 `instance_name` 字段得到**

1.	编辑虚拟机的 XML 文件，使用如下命令

	```console
	# virsh edit old_instance_domain
	```

	在现有 `<disk>...</disk>` 标签段后再添加如下代码段，以添加光驱设备并挂载 iso:

	```xml
	<disk type='file' device='cdrom'>
	<driver name='qemu' type='raw'>
	<source file='/home/openstack/rhel-server-5.3-x86_64-dvd.iso'>
	<target dev='hdc' bus='ide'>
	<readonly>
	</disk>
	```

	再在 `<os>...</os>` 标签内，`<boot dev='hd'>` 之前添加一行：

	```xml
	<boot dev='cdrom'>
	```

2.	此时再将该实例 `destroy` 再 `start` 就会从光盘启动了

	```shell-session
	# destroy old_instance_domain
	# start old_instance_domain
	```

## 延伸操作: 救援模式自动恢复 `/proc` 目录

在救援模式修复文件系统后，还可能会遇到文件丢失的情况，本次还遇到了 `/etc` 与 `/proc` 目录丢失的情况，导致无法启动的情况，将修复过程关键点总结如下：

-	当缺少 `/etc` 目录时，虚机无法启动，此时可以进入救援模式从其他 os 拷贝
-	当缺少 `/etc`，或其他诸如 `/usr/`, `/var` 等目录时，Linux Rescue Mode 模式无法找到有效的 Linux 分区，即系统文件丢失了:


	![找不到有效的 Linux 分区](http://static-hiaero.b0.upaiyun.com/main/wp-content/uploads/2013/02/cant_find_linux_part2.jpg)

	当系统文件齐全时，进入 Linux Resuce Mode 的提示是这样的：

	![找到了有效的 Linux 分区](http://static-hiaero.b0.upaiyun.com/main/wp-content/uploads/2013/02/found_linux_part2.jpg)

-	当 `/proc` 目录丢失时，操作系统无法启动，恢复办法为，在**可以找到有效 Linux 分区**的情况下，进入一次Linux Rescue Mode 即可自动修复
