title: 小把戏在 Mac Terminal 下安全便捷的登录服务器
date: 2013-12-31 08:45
categories: Tech Logs
tags:
- terminal
- Mac
---

Mac 下，可以直接使用自带的 Terminal 软件来登录服务器进行运维操作，而无须在 SecureCRT 的盗版与昂贵之间纠结。

Terminal 没有保存密码登功能，每次登录服务器都要手动输入密码很不方便，但是如果在普通用户下建立 Mac 到 Server 的 ssh 信任的话又很不安全。即使建立信任在 root 用户下了，切换过去的话还是很不方便，并且每次想 `commadn+n` 建立新的 session 都要重新 `su - root`。总之就是没法直接在普通用户下方便的 ssh 到服务器而避免安全问题。

所以这里耍一个小把戏

1.	在 root 用户之下建立从 Mac 到 Server 的信任
2.	在普通用户下建立如下的几个 alias：

	```bash
	alias sudo='sudo ' #注意此处sudo后面有一个空格
	alias server01='ssh root@IP_server01'
	alias server02='ssh root@IP_server02'
	```

3.	在普通用户下就可以使用 `sudo server01` 来登录服务器了

**其中 `alias sudo='sudo '` 的解释：**

在 Unix/Linux 中，`sudo` 命令是忽略任何 alias 的，但是 Bash 又一个功能，如果 alias 的末尾是一个空格的话，那么在解析 alias 的时候会将之后的命令字符也当作 alias 来解析，这个地方如果不设置 `alias sudo='sudo '` 的话，`sudo server01` 是不会生效的，所以 bash 的这个功能是这个小把戏的精髓所在。

**Enjoy your trick~**
