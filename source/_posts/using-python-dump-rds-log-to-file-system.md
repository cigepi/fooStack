title: 使用 Python 将 Amazon RDS 日志连续转储到本地磁盘
date: 2013-12-07 11:04
categories: Tech Logs
tags:
- Python
- AWS
- RDS
---

Amazon RDS（MySQL Engine）提供了将 MySQL Engine 的慢查询日志存储到数据表或数据文件的功能。

当我们需要对慢查询日志进行分析时，如果该日志是存储在数据表中的，就需要为其编写复杂的 SQL 语句来分析，带来的困扰是，第一编写 SQL 语句很麻烦，第二执行这些 SQL 会为数据库带来负载，所以请尽量将 RDS（MySQL Engine）的日志存储设置为数据文件（ `log_output = FILE`）。

而这些存储到数据文件中的 RDS Logs 又有以下特点：

-	对于 RDS 来说，用户是没有文件系统权限的，所以虽然设置为存储到数据文件，但是实际上你还是必须使用 API 来访问这些文件
-	只保存最近 24 个小时内的日志
-	日志每小时轮替一次，共有 24 个文件
-	24 个日志文件单从文件名无法判断先后顺序，并且没有按时间排序列出这些文件的功能，使用 Shell 很难一个循环简单按序 Dump
-	API 有 Web 管理页面，及 RDS Command Line Tools

据其特点，实际使用中会遇到的问题是：

-	24 小时之前的性能问题就追踪不到了
-	日志使用 API 即时获取，因此没有办法用 mysqlslap/pt-query-digest 等工具进行分析
-	一天的日志被分为 24 个，很难一次操作

为了解决以上问题，我做了一个小工具 [rds_dump_log](https://github.com/haiyangpeng/dump_rds_log)，用来放在 Linux 的 crontab 中，每天从 RDS API 获取日志文件，按序 Dump 到位于本地磁盘的单个日志文件中。

## 配置方法：

1.	首先，你需要在你的能访问 RDS 的 Linux Server 上安装配置 [RDS Command Line Tools](http://docs.aws.amazon.com/AmazonRDS/latest/CommandLineReference/StartCLI.html)，这是一个 JAVA 编写的命令行工具，可以在 Linux 中使用命令行来管理 RDS。

2.	将以下环境变量添加到你的 `/etc/profile` 文件中并 `source`（各变量值根据你的实际安装路径配置）。

```
# Settings for AWS Command Line Tools
export AWS_RDS_HOME='/home/dba/maintain/RDSCli-1.15.001'
export JAVA_HOME='/usr/java/default'
export PATH=${PATH}:${AWS_RDS_HOME}/bin
export EC2_REGION=us-east-1
export AWS_CREDENTIAL_FILE=/home/dba/maintain/RDSCli-1.15.001/credential-file-path.template
```

3.	在 Shell 中使用 `rds --help` 来测试你的工具是否安装成功。

4.	安装 Python 2.7 以上版本（如果现有版本已经 >=2.7，则跳过）。

5.	下载 [dump_rds_log.py](https://github.com/haiyangpeng/dump_rds_log/blob/master/dump_rds_log.py)

6.	添加定时任务，每日自动 Dump 日志文件到本地

需要注意的是，由于该日志每小时自动轮替，并且只保存最近 24 小时的文件，因此你的 crontab 的两次连续任务之间的间隔必须小于 24 小时，不然你就会漏掉一些日志了，我选择的是每 12 小时 Dump 一次。

另外，[rds_dump_log](https://github.com/haiyangpeng/dump_rds_log) 的使用方法可用 `python dump_rds_log.py --help` 查看，或到[这里查看](https://github.com/phang001/dump_rds_log/blob/master/README.md)

例如，我当前配置的自动 Dump GameFuse 的 slow 日志的定时任务如下：

```
###将RDS的SLOW LOG DUMP到本地存放#######################
0 0,12 * * * source /etc/profile; /usr/bin/python /home/slow_log_dump/dump_rds_log.py rds-id-1 slow /home/slow_log_dump/
0 1,13 * * * source /etc/profile; /usr/bin/python /home/slow_log_dump/dump_rds_log.py rds-id-2 general /home/general_log_dump/
0 2,14 * * * source /etc/profile; /usr/bin/python /home/slow_log_dump/dump_rds_log.py rds-id-3 slow /home/slow_log_dump/
```

**注意：**

1.	crontab 命令中的 `source /etc/profile` 是必须的，因为 Linux crontab 目前不能自动读取到系统环境变量，而调用 RDS Command Line Tools 需要上述那些环境变量的支持。
2.	虽然 AWS 提供了一个 python 库叫做 `boto`，用来管理 RDS，但是目前该库功能有限，远不如 RDS Command Line Tools 功能强大，因此随着 boto 的进步，这个工具可能会变得更简单易用。
3.	实际上该工具也可以用来转储 general log、error log，将第二个参数改为 `general` 或 `error` 就可以了，甚至也可用用来转储 RDS 的 sqlserver engine、oracle engine 的日志。
