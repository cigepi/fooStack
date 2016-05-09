title: DokuWiki 简介
date: 2013-08-08 01:00
categories: Tech Logs
tags:
- writing
- Wiki
---

**为了能更好的管理具有关联性的知识块，我搭建了一个个人 [WIKI](http://wiki.hiaero.net/)。**

使用了 [DokuWiki](https://www.dokuwiki.org/dokuwiki) 这款开源 Wiki 引擎，DokuWiki 具有简洁、扩展性强、免数据库等特点，当然我选择它最重要的原因是喜欢其 UI 风格。

## Wiki 与 DokuWiki

相比于单纯使用 Blog，使用 Wiki 来进行个人知识管理具有如下特点：

-	Wiki 中每创建一个页面，必须在已存在的页面中有一个链接指向它，因此 Wiki 中没有孤岛页面
-	Wiki 注重“索引”与“结构”，而 Blog 注重“记录”
-	Wiki 依靠必选的“链接”来关联页面，而 Blog 依靠“分类目录”与 tag 来关联博文。Wiki 关联性更强知识呈“网状”，Blog 中的知识呈“片状散落”
-	Wiki 突出内容本身，一切功能为内容服务，舍弃了其他冗杂
-	Wiki 会追踪每个页面的历史记录，类似 git
-	Wiki 的采用纯文本存储正文，使用特有的标记语法来编写文档，全文中没有 tab!
-	Wiki 的语法叫做 Wikitext，虽然每个不同 Wiki 引擎语法略有不同，但大体相似
-	Wikitext 虽然简单，却最适合编写文档，使用 Wikitext 写出来的文档具有结构清晰、风格统一、源码易读易维护的优点

DokuWiki 作为一款开源 Wiki 引擎，具有下面这些特点：

-	崇尚 *It's better when it's simple*
-	使用 PHP 实现
-	无需数据库支持
-	具有 ACL 功能
-	扩展性好，目前有 958 款插件，109 款皮肤（或称模板，大多丑得哭）
-	UI 简洁靓丽

## DokuWiki 的安装

将下载好的软件包上传到你的 Web 目录中并解压，浏览器访问 `http://yourdomain/path_to_dokuwiki/install.php`，根据提示进行即可

### DokuWiki 替换 logo 及 favicon

个人 Wiki 搭建好之后，最碍眼的莫过于打开自己的网站却显示着别人的 logo了。要替换 logo 只需替换图片 `/path_to_dokuwiki/lib/tpl/dokuwiki/images/logo.png` 即可

favicon 即显示在浏览器标签、收藏夹中的你的网站图标。要替换 favicon 只需替换图标 `/path_to_dokuwiki/lib/tpl/dokuwiki/images/favicon.ico` 即可

## 进一步学习 DokuWiki

要进一步学习 DokuWiki 的使用方法，请参考[官方手册](https://www.dokuwiki.org/manual)，非常详细与友好。

[^Guido]
