title: OpenStack Grizzly ReleaseNotes 的 Nova部分翻译
date: 2013-05-15 21:59
categories: Tech Logs
tags:
- OpenStack
---

尝试翻译了 OpenStack Grizzly ReleasNotes 中关于 Nova 关键新特性的描述，以总结、归档为目的，并锻炼英转中的表述能力，[原文地址](https://wiki.openstack.org/wiki/ReleaseNotes/Grizzly)

## OpenStack Compute (Nova)

### Key New Features

-	**Cells:** Grizzly will include a preview (experimental) release of cells functionality. Cells provides a new way to scale nova deployments, including the ability to have compute clusters (cells) in different geographic locations all under the same nova API. See [Cell Docs](http://docs.openstack.org/trunk/openstack-compute/admin/content/ch_cells.html) for more details.
-	**Cells:** G版本将会包含 Cells 功能的预览版。Cells 使 OpenStack 具有在统一的 API 下将不同的计算集群分布部署到不同地理位置的能力，增强了 Nova 部署的可扩展性。

---

-	**Availability Zones:** Availability Zone support has been enhanced. Previously, the only way to set the availability zone for a given compute node was via its configuration file. You can now set a node's availability zone via the API.
-	**Availability Zones:** G 版本加强了对 Availability Zone 的支持。在之前，为一个计算节点指定 Availability Zone 的唯一方式是使用配置文件，但是现在可以通过 API 来完成了。

---

-	**Admin APIs:** There have been multiple additions to the API for administrative actions. This has been done to continue to move away from needing the nova-manage utility for most administrative tasks.
-	**Admin APIs:** G 版本为增加了大量用作 OpenStack 管理操作的 API，使对 OpenStack 对 nova-manage 的依赖在持续减少。

---

-	**API support for instance passwords:** This enhancement to nova improves support for instances that require passwords to work, such as those running Windows. Instances can now generate and post an encrypted password to the metadata API (write once). This password can be retrieved via the public nova API. This functionality can be integrated with a guest initialization tool such as cloud-init.
-	**API support for instance passwords:** G 版本更好的支持了那些需要密码进行工作的虚拟机，例如 windows。实例可以生成并将密文密码送入元数据 API 中（只可写一次），并且在之后可以通过 API 获取到这个密码。这个功能可以和 colud-init 之类的实例初始化工具整合。

---

-	**Bare metal provisioning:** Grizzly includes a new hypervisor driver than can deploy machine images to bare metal, allowing tasks to run with no virtualisation overhead. This is supported but not fully featured - see the hypervisor feature matrix for details. Also, in the event one wishes (or needs) to customize the machine image being deployed, the diskimage-builder project on stackforge is recommended. See [bare metal docs](https://wiki.openstack.org/wiki/Baremetal) for more details.
-	**Bare metal provisioning:** G 版本新的 hypervisor 驱动可以将机器镜像部署至裸机，更进一步可以使其运行在没有虚拟化支持的机器上。而这并不是全部，更为详细的信息请查看 hypervisor feature matirx。同时，对于那些想自定义机器的镜像的用户，推荐 diskimage-builder 项目。

---

-	**Improved MySQL connector performance:** Some enhancements have been made to allow better interaction with MySQL and the threading model used by nova (eventlet).
-	**Improved MySQL connector performance:** 对 Nova 使用的 MySQL 连接器的（eventlet）做了加强，使其可以与 MySQL 更好的交互并拥有更好的线程模型。

---

-	**Database archiving:** Support for pruning deleted items and placing them in separate tables to keep the most frequently written tables from growing without bounds.
-	**Database archiving:** 将 MySQL 数据库中那些被标记删除的条目放入独立的、专门的表中进行归档，防止那些被频繁写的表无限增长。

---

-	**Instance Action Tracking:** Nova has been updated to keep track of all actions performed on an instance. There is an API extension for accessing this information. Viewing the list of instance actions provides deeper insight into the history of an instance. It also provides much better error reporting for users and administrators.
-	**Instance Action Tracking:** Nova 现在可以追踪在某个实例上执行的所有动作，并有一个 API 扩展可以访问这些信息。还可以通过实例行为列表来深入了解某个实例的动作历史记录。最后，对用户与管理员来会说，将会得到更好的错误报告。

---

-	**No-DB-Compute:** The nova-compute service can optionally run in a mode where it has no direct access to the database. This improves Nova's security, though some concerns have been raised about the performance of this new mode.
-	**No-DB-Compute:** 用户可以选择将 nova-compute 服务工作在不直接访问数据库的模式下. 这样可以提高 nova 的安全性，同时也会带来了一丝对性能的担忧。

---

-	**Quantum Security Groups Proxy:** When managing security groups through Nova's API, all actions will be proxied to Quantum when Quantum is the network provider.
-	**Quantum Security Groups Proxy:** 在运用 Quantum 的环境中，当通过 nova 的 API 来进行安全组管理时，所以的操作都会被代理至 Quantum 中执行。

---

-	**File injection without mounting guest filesystem:** Nova has the ability to use libguestfs to support file injection into a guest filesystem. Previously this was done by mounting the guest filesystem on the host. This has been refactored to use libguestfs APIs that do not require mounting the guest filesystem, which is much more secure.
-	**File injection without mounting guest filesystem:** Nova 可以使用 libguestfs 来将一个文件注入到某个实例的文件系统中，而不需要在 host 上挂载这个实例的文件系统，这样对实例来说将更加安全。

---

-	**Default Security Group Rules:** Nova can now be made to add rules to the default security group when it is created for a tenant.
-	**Default Security Group Rules:** Nova 现在可以在为某个租户创建默认安全组的同时加入规则了。

---

-	**libvirt Custom Hardware:** The libvirt driver in Nova will now check for properties on an image that specify specific hardware types that should be used. An example of when this is useful is for an image that does not support virtio, and should use a fully virtualized hardware type instead.
-	**libvirt Custom Hardware:** libvirt 驱动现在可以检测出某个镜像所需要的特定硬件类型。这个特性的一个实用场景是：当某个镜像不支持 virtio，不得不使用全部使用虚拟化硬件时。

---

-	**libvirt Spice Console:** The libvirt driver in Nova now supports Spice virtual consoles.
-	**libvirt Spice Console:** libvirt 驱动现在支持 Spice（vnc之外another一个图形控制台）了。

---

-	**powervm Resize, Migrate, and Snapshot:** The powervm driver in Nova now supports the resize, migrate and snapshot operations.
-	**powervm Resize, Migrate, and Snapshot:** nova 现在支持 pwervm（据 google 是 IBM 的一种虚拟化产品）的 resize, migrate 和 snapshot 操作。

---

-	**VMware Driver Improvements:** Several improvements were made to the VMware driver, including support for VNC consoles, iSCSI volumes, live migration, rescue mode, Quantum, and improved Glance integration (OVF support, better download performance).
-	**VMware Driver Improvements:** 对 VMware 驱动的支持做出了几项改进，包括支持 VNC consoles, iSCSI volumes, live migration, rescue mode, Quantum 并增强了与 Glance 的整合（支持 OVF 与更好的镜像下载性能）。

---

-	**Unique Instance Naming:** When issuing an API command to create multiple servers, Nova will now give each instance a unique name based on a configured template. Previously all instances would have the same name.
-	**Unique Instance Naming:** 当发出一个 API 命令来创造多个实例时，nova 现在会按照提前配置的模板为每个实例分配一个唯一的名字。在这之前，这些名字都将是相同的。

---

-	**Availability Zones in OpenStack API:** Support for availability zones has been enhanced in the OpenStack API. You can now list availability zones through the API. The availability zone for an instance is also included in instance details.
-	**Availability Zones in OpenStack API:** 增强对 availability zones 的支持，可以使用 API 来列出所有的 availability zones，并且在实例详细信息中也将显示其所属的 availability zones。

---

-	**Glance Direct Image File Copy:** If Glance provides Nova a URL to the image location on a shared filesystem, Nova will now get the image content from there instead of through the Glance API. This will result in faster instance boot times under some circumstances.
-	**Glance Direct Image File Copy:** 如果 glance 为 nova 提供一个共享文件系统中的镜像地址， nova 将会直接从那里获取镜像而不是 Glance API，这将会提高在某些环境下实例的启动速度。

---

-	**Boot without image:** It is now possible to boot a volume-backed instance without specifying an image, if block-device-mapping is passed to the nova boot command.
-	**Boot without image:** 现在可以不用指定镜像，而通过在 nova boot 命令中传递 block-device-mapping 选项来启动一个基于volume的实例。

---

-	**Quota-instance-resource:** It is now possible to set accurate quota for CPU, disk IO, and network interface bandwidth(duo to a bug bandwidth Qos can't works) of an instance. By using this feature, you can provide a consistent amount of CPU capacity no matter what the actual underlying hardware.
-	**Quota-instance-resource:** 现在可以为 CPU、磁盘 I/O、网络接口带宽（因为一个带宽 bug，QoS 无法工作（是这么翻么？？！！））设置精确的配额。通过这个特性，你可以为实例提供更加一致的性能，而不论底层硬件的实际情况。

---

-	**Network adapter hot-plug:** It is now possible to hot-plug a pre created port to a running instance.
-	**Network adapter hot-plug:** 现在可以为一个运行的实例热插拔网络适配器了。

---

-	**Quotas for fixed IP addresses:** It is now possible to set quotas for the allocation of fixed IP addreses (set quota_fixed_ips in nova.conf - default is unlimited)
-	**Quotas for fixed IP addresses:** 现在可以为 fixed IP 的分配设置配额了（在 nova.conf 中设置quota_fixed_ips，默认是没有限制的）
