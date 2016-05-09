title: 速度更快的海外 VPS
date: 2014-05-17 10:21
categories: Tech Logs
tags:
- DigitalOcean
- Linode
- vpn
- vps
---

update 2014-05-17:

T_T，忘记考虑价格了 DigitalOcean 是 $5/月，Linode 是 $20/月，请愉快的选择 DigitalOcean.

<hr>

目前 geekers 会选的两家主流 VPS 供应商应该是 DigitalOcean 与 Linode，测了两家不同机房从北京的访问速度。

Linode 我只测了一个日本机房，DigitalOcean 测了 3 个，还对比了公司在加州的 IDC，结果如下：

-	最快的是 Linode 日本，平均延迟在 130-170ms，但丢包率 0.1% 以下
-	最烂的是 DigitalOcean 新加坡，虽然速度堪比日本，丢包率达到 7%
-	其他为 DigitalOcean 纽约、三藩、阿姆斯特丹，速度都在 250ms 上下，丢包率 0.1% 以下
-	公司加州 idc，与 DigtialOcean 三藩/纽约相当

买了一个 VPS 之后最重要的两件事是什么？

1.	搭建一个 blog
2.	搭建一个 vpn 翻跃长城

基于 2，请选择最快，连接最稳定的 VPS 吧。
