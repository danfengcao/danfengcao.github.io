---
layout: post
title:  binlog2sql：从MySQL binlog得到你要的SQL
date:   2016-12-07 19:45:00
comment: true
category: "MySQL"
---

从MySQL binlog得到你要的SQL。根据不同设置，你可以得到原始SQL、回滚SQL、去除主键的INSERT SQL等。

用途
===========

* 数据回滚
* 主从切换后数据不一致的修复
* 从binlog生成标准SQL，带来的衍生功能


安装
==============

{% highlight bash %}

$ git clone https://github.com/danfengcao/binlog2sql.git
$ pip install -r requirements.txt

{% endhighlight %}

使用
=========

### MySQL server必须设置以下参数:

    [mysqld]
    server-id = 1
    log_bin = /var/log/mysql/mysql-bin.log
    max_binlog_size = 1000M
    binlog-format = row

###基本用法

**解析出标准SQL**

{% highlight bash %}
$ python binlog2sql.py -h127.0.0.1 -P3306 -uadmin -p'admin' -d dbname -t table1 table2 --start-file='mysql-bin.000002' --start-pos=1240

输出：
INSERT INTO d(`did`, `updateTime`, `uid`) VALUES (18, '2016-12-07 14:01:14', 4);
INSERT INTO c(`id`, `name`) VALUES (0, 'b');
UPDATE d SET `did`=17, `updateTime`='2016-12-07 14:01:14', `uid`=4 WHERE `did`=18 AND `updateTime`='2016-12-07 14:01:14' AND `uid`=4 LIMIT 1;
DELETE FROM c WHERE `id`=0 AND `name`='b' LIMIT 1;

{% endhighlight %}

**解析出回滚SQL**

{% highlight bash %}
$ python binlog2sql.py --flashback -h127.0.0.1 -P3306 -uadmin -p'admin' -d dbname -t table1 table2 --start-file='mysql-bin.000002' --start-pos=1240

输出：
INSERT INTO c(`id`, `name`) VALUES (0, 'b');
UPDATE d SET `did`=18, `updateTime`='2016-12-07 14:01:14', `uid`=4 WHERE `did`=17 AND `updateTime`='2016-12-07 14:01:14' AND `uid`=4 LIMIT 1;
DELETE FROM c WHERE `id`=0 AND `name`='b' LIMIT 1;
DELETE FROM d WHERE `did`=18 AND `updateTime`='2016-12-07 14:01:14' AND `uid`=4 LIMIT 1;
{% endhighlight %}

###选项
**mysql连接配置**

-h host; -P port; -u user; -p password

**解析模式**

--realtime 持续同步binlog。可选。不加则同步至执行命令时最新的binlog位置。

--popPk 对INSERT语句去除主键。可选。

-B, --flashback 生成回滚语句。可选。与realtime或popPk不能同时添加。

**解析范围控制**

--start-file 起始解析文件。必须。

--start-pos start-file的起始解析位置。可选。默认为start-file的起始位置；

--end-file 末尾解析文件。可选。默认为start-file同一个文件。若解析模式为realtime，此选项失效。

--end-pos end-file的末尾解析位置。可选。默认为end-file的最末位置；若解析模式为realtime，此选项失效。

**对象过滤**

-d, --databases 只输出目标db的sql。可选。默认为空。

-t, --tables 只输出目标tables的sql。可选。默认为空。

###应用案例

**主从切换后数据不一致的修复**，详细描述可参见[github.com/danfengcao/binlog2sql/blob/master/example/FixOldMasterExtraData.md](github.com/danfengcao/binlog2sql/blob/master/example/FixOldMasterExtraData.md)

1、提取old master未同步的数据，并对其中的insert语句去除主键（为了防止步骤3中出现主键冲突）

{% highlight bash %}
$ python binlog2sql.py --popPk -h10.1.1.1 -P3306 -uadmin -p'admin' --start-file='mysql-bin.000040' --start-pos=125466 --end-file='mysql-bin.000041' > oldMaster.sql
{% endhighlight %}

2、将old master回滚，开启同步。同步正常；

{% highlight bash %}
$ python binlog2sql.py --flashback -h10.1.1.1 -P3306 -uadmin -p'admin' --start-file='mysql-bin.mysql-bin.000040' --start-pos=125466 --end-file='mysql-bin.000041' | mysql -h10.1.1.1 -P3306 -uadmin -p'admin'
{% endhighlight %}

3、在new master重新导入改造后的sql；

{% highlight bash %}
$ mysql -h10.1.1.2 -P3306 -uadmin -p'admin' < oldMaster.sql
{% endhighlight %}

###限制
* mysql server必须开启，离线模式下不能解析binlog
* binlog格式必须是行模式
* flashback模式只支持DML，DDL将不做输出
* flashback模式，一次性处理的binlog不宜过大，不能超过内存大小(有待优化)
* 目前已测试环境
    * Python 2.7
    * MySQL 5.6

###优点（对比mysqlbinlog）
* 纯Python开发，安装与使用都很简单
* 自带flashback、popPk解析模式，无需再装补丁
* 解析为标准SQL，方便理解、调试
* 代码容易改造，可以支持更多个性化解析

###联系我
有任何问题，请与我联系 [danfengcao.info@gmail.com](danfengcao.info@gmail.com)



参考资料
==============
[1] 彭立勋, [MySQL下实现闪回的设计思路](http://www.penglixun.com/tech/database/mysql_flashback_feature.html)

[2] \_\_七把刀__, [MySQL binlog格式解析](http://www.jianshu.com/p/c16686b35807?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io)

[3] noplay, [Pure Python Implementation of MySQL replication protocol build on top of PyMYSQL](https://github.com/noplay/python-mysql-replication)

