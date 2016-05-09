title: 无状态、REST、RESTful 和 Web Services
date: 2013-05-25 11:08
categories: Tech Logs
tags:
- REST
- RESTful
---

在理解 OpenStack 的过程中，常常遇到 REST 这个概念，现从各处搜罗如下：


## 对 Web Service 的理解：

Web 服务有点像对计算机友好的网页，基于让程序可以跨网络交换信息的标准和协议，一般是使用一个程序，也就是客户机或服务请求者(serice requester)请求信息或服务；使用另外一个程序，也就是服务器或服务器提供者(service provider)提供信息或服务。

它是在另外一个层次上的 Web 分析，或者可以将 Web 服务看作为用计算机控制客户机而不是人类——所设计的动态网页。

来自 [Python基础教程](http://www.amazon.cn/Python%E5%9F%BA%E7%A1%80%E6%95%99%E7%A8%8B-%E8%B5%AB%E7%89%B9%E5%85%B0/dp/B003TSBAMM/ref=sr_1_1?ie=UTF8&qid=1369390044&sr=8-1&keywords=python%E5%9F%BA%E7%A1%80%E6%95%99%E7%A8%8B)

## 无状态协议：

无状态服务器是指一种把每个请求作为与之前任何请求都无关的独立的事务的服务器。

HTTP 服务器就是一个例子。以 URL 形式提交的客户端请求可能包含 cookies 等带状态的数据，这些数据完全指定了所需的文档，而不需要其他之前请求的上下文或内存。

来自 [WikiPedia](http://zh.wikipedia.org/wiki/%E6%97%A0%E7%8A%B6%E6%80%81%E5%8D%8F%E8%AE%AE)

## REST

REST (REpresentation State Transfer，表述性状态转移) 。REST 指的是一组架构约束条件和原则。REST 从资源的角度来观察整个网络，分布在各处的资源由 URI 确定，而客户端的应用通过 URI 来获取资源的表征。REST 是设计风格而不是标准。REST 通常基于使用 HTTP，URI，和 XML 以及 HTML 这些现有的广泛流行的协议和标准。

### 一些特征：


-	资源是由URI来指定。
-	对资源的操作包括获取、创建、修改和删除资源，这些操作正好对应HTTP协议提供的标准方法GET、POST、PUT和DELETE。
-	通过操作资源的表现形式来操作资源。
-	为所有"事物"定义ID
-	将所有事物链接在一起
-	资源多重表述
-	无状态通信

### REST中的资源：

在服务器端，应用程序状态和功能可以分为各种资源。资源是一个有趣的概念实体，它向客户端公开。资源的例子有：应用程序对象、数据库记录、算法等等。每个资源都使用 URI (Universal Resource Identifier) 得到一个惟一的地址。

### REST中的无状态：

Web 应用程序最重要的 REST 原则是，客户端和服务器之间的交互在请求之间是无状态的。从客户端到服务器的每个请求都必须包含理解请求所必需的信息。如果服务器在请求之间的任何时间点重启，客户端不会得到通知。此外，无状态请求可以由任何可用服务器回答，这十分适合云计算之类的环境。客户端可以缓存数据以改进性能。

来自 [WikiPedia](http://zh.wikipedia.org/wiki/REST), [百度百科](http://baike.baidu.com/view/1077487.htm), [infoQ](http://www.infoq.com/cn/articles/rest-introduction)

### RESTful:

满足 REST 的应用程序或设计就是 RESTful。
