title: MySQL 服务器 Raid 卡 Cache 策略的设置总结
date: 2013-05-10 12:05
categories: Tech Logs
tags:
- MySQL
- Raid
---


1.	开启 `Write Back`，提高写效率，详细请看[这里](http://benjr.tw/node/682)

	```console
​	# MegaCli -LDSetProp -WB -LALL -aALL
	```

2.	开启 `Bad BBU Write Back`

	-	默认情况下，如果 Raid 卡电池坏掉，Raid 卡会自动将 `Write Back` 切换到 `Write Through`，这个时候 I/O 就会变慢
	-	开启 `Bad BBU Write Back` 的情况下，如果电池坏掉，那么 I/O 依然得到保障，但是带来了丢失数据的风险。假如此时机器断电，而 Cache 中的数据没有电池（BBU）的保护，数据就丢失了。
	-	相比于机器断电，Raid 卡电池充放电会更常见，因此开启 `Bad BBU Write Back` 保证 I/O 速度。
	-	以上内容来自[这里](http://www.hellodb.net/2011/07/mysql-linux-hardware-tuning.html)

	```console
	# MegaCli -LDSetProp -CachedBadBBU-LALL -aALL
	```

3.	关闭读操作使用 Cache

	因为 Raid 卡 Cache 容量有限，为了保证写 Cache 的使用，因此关闭读 Cache

	```console
	# MegaCli -LDSetProp -Direct -LAll -aAll
	```

4.	关闭磁盘本身 Cache

	因为使用 Raid 卡 Cache，因此关闭磁盘 Cache

	```console
	# MegaCli -LDSetProp -DisDskCache -Lall -aALL
	```

5.	开启 `Adaptive ReadAhead`

	-	`ReadAhead` 是预读，而预读仅仅对顺序磁盘 I/O 有性能提升，因此将其关闭。
	-	`Adaptive ReadAhead` 是自适应读，自动决定是否预读，详细请看[这里](http://tech.foolpig.com/2011/08/31/raid-cache-policy/)

	```console
	# MegaCli -LDSetProp ADRA -LALL -aALL
	```

6.	按照以上描述设置后，Dell 服务器的 `Current Cache Policy` 理应如下：       

	```console
	# MegaCli -LDGetProp -Cache -LALL -aALL
	Adapter 0-VD 0(target id: 0): Cache Policy:WriteBack, ReadAdaptive, Direct, Write Cache OK if bad BBU
	```

	从左至右逗号隔开的次，分别对应上文中的 1，5，3，2 的设置结果

	```console
	# MegaCli -LDGetProp -DskCache -LALL -aALL
	Adapter 0-VD 0(target id: 0): Disk Write Cache : Disabled
	```

	此为上文中 4 的设置结果
