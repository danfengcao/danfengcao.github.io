---
layout: post
title:  MySQL误操作后如何快速恢复数据
date:   2016-12-09 20:15:00
comment: true
category: "MySQL"
---

基本上每个跟数据库打交道的程序员（当然也可能是你同事）都会碰一个问题，MySQL误操作后如何快速回滚？比如，不小心update了整张表的某个字段，或者delete一张表，忘加限制条件，整张表都没了。假如这还是线上环境核心业务数据，那这事就闹大了。误操作后，能快速回滚数据是非常重要的。


传统解法
===
用全量备份重搭实例，再利用增量binlog备份，恢复到误操作之前的状态。然后跳过误操作的SQL，再继续应用binlog。对于DML的回滚，此法费时费力，不值得再推荐。

利用binlog2sql快速闪回
===
首先，确认你的MySQL server开启了binlog，设置了以下参数:

	[mysqld]
	server-id = 1
	log_bin = /var/log/mysql/mysql-bin.log
	max_binlog_size = 100M
	binlog-format = row

如果没有开启binlog，也没有预先生成回滚SQL，那真的无法快速回滚了。对存放重要业务数据的MySQL，强烈建议开启binlog。

随后，安装开源工具[binlog2sql](https://github.com/danfengcao/binlog2sql)。binlog2sql是一款简单易用的binlog解析工具，其中一个功能就是生成回滚SQL。

{% highlight bash %}
shell> git clone https://github.com/danfengcao/binlog2sql.git
shell> pip install -r requirements.txt
{% endhighlight %}

然后，我们就可以生成回滚SQL了。

**背景**：小明在20点多时误删了test库tbl表整张表的数据，需要紧急回滚。

{% highlight bash %}
test库tbl表原有数据
mysql> select * from tbl;
+----+--------+---------------------+
| id | name   | addtime             |
+----+--------+---------------------+
|  1 | 小赵   | 2016-12-10 00:04:33 |
|  2 | 小钱   | 2016-12-10 00:04:48 |
|  3 | 小孙   | 2016-12-13 20:25:00 |
|  4 | 小李   | 2016-12-12 00:00:00 |
+----+--------+---------------------+
4 rows in set (0.00 sec)

mysql> delete from tbl;
Query OK, 4 rows affected (0.00 sec)

20:28时，tbl表误操作被清空
mysql> select * from tbl;
Empty set (0.00 sec)
{% endhighlight %}

**恢复数据步骤**：

1. 登录mysql，查看目前的binlog文件

{% highlight bash %}
mysql> show master status;
+------------------+-----------+
| Log_name         | File_size |
+------------------+-----------+
| mysql-bin.000051 |       967 |
| mysql-bin.000052 |       965 |
+------------------+-----------+
{% endhighlight %}

2. 最新的binlog文件是mysql-bin.000052，我们再定位误操作SQL的binlog位置。误操作人只能知道大致的误操作时间，我们根据大致时间过滤数据。

{% highlight bash %}
shell> python binlog2sql/binlog2sql.py -h127.0.0.1 -P3306 -uadmin -p'admin' -dtest -ttbl --start-file='mysql-bin.000052' --start-datetime='2016-12-13 20:25:00' --stop-datetime='2016-12-13 20:30:00'
输出：
INSERT INTO `test`.`tbl`(`addtime`, `id`, `name`) VALUES ('2016-12-13 20:25:00', 3, '小孙'); #start 4 end 290 time 2016-12-13 20:25:46
INSERT INTO `test`.`tbl`(`addtime`, `id`, `name`) VALUES ('2016-12-13 20:26:00', 4, '小李'); #start 317 end 487 time 2016-12-13 20:26:26
UPDATE `test`.`tbl` SET `addtime`='2016-12-12 00:00:00', `id`=4, `name`='小李' WHERE `addtime`='2016-12-13 20:26:00' AND `id`=4 AND `name`='小李' LIMIT 1; #start 514 end 701 time 2016-12-13 20:27:07
DELETE FROM `test`.`tbl` WHERE `addtime`='2016-12-10 00:04:33' AND `id`=1 AND `name`='小赵' LIMIT 1; #start 728 end 938 time 2016-12-13 20:28:05
DELETE FROM `test`.`tbl` WHERE `addtime`='2016-12-10 00:04:48' AND `id`=2 AND `name`='小钱' LIMIT 1; #start 728 end 938 time 2016-12-13 20:28:05
DELETE FROM `test`.`tbl` WHERE `addtime`='2016-12-13 20:25:00' AND `id`=3 AND `name`='小孙' LIMIT 1; #start 728 end 938 time 2016-12-13 20:28:05
DELETE FROM `test`.`tbl` WHERE `addtime`='2016-12-12 00:00:00' AND `id`=4 AND `name`='小李' LIMIT 1; #start 728 end 938 time 2016-12-13 20:28:05
{% endhighlight %}

3. 我们得到了误操作sql的准确位置在728-938之间，再根据位置过滤，使用flashback模式生成回滚sql，检查回滚sql是否正确(注：真实环境下，此步经常会进一步筛选出需要的sql。结合grep、编辑器等)

{% highlight bash %}
shell> python binlog2sql/binlog2sql.py -h127.0.0.1 -P3306 -uadmin -p'admin' -dtest -ttbl --start-file='mysql-bin.000052' --start-position=3346 --stop-position=3556 -B > rollback.sql | cat
输出：
INSERT INTO `test`.`tbl`(`addtime`, `id`, `name`) VALUES ('2016-12-12 00:00:00', 4, '小李'); #start 728 end 938 time 2016-12-13 20:28:05
INSERT INTO `test`.`tbl`(`addtime`, `id`, `name`) VALUES ('2016-12-13 20:25:00', 3, '小孙'); #start 728 end 938 time 2016-12-13 20:28:05
INSERT INTO `test`.`tbl`(`addtime`, `id`, `name`) VALUES ('2016-12-10 00:04:48', 2, '小钱'); #start 728 end 938 time 2016-12-13 20:28:05
INSERT INTO `test`.`tbl`(`addtime`, `id`, `name`) VALUES ('2016-12-10 00:04:33', 1, '小赵'); #start 728 end 938 time 2016-12-13 20:28:05
{% endhighlight %}

4. 确认回滚sql正确，执行回滚语句。登录mysql确认，数据回滚成功。

{% highlight bash %}
shell> mysql -h127.0.0.1 -P3306 -uadmin -p'admin' < rollback.sql

mysql> select * from tbl;
+----+--------+---------------------+
| id | name   | addtime             |
+----+--------+---------------------+
|  1 | 小赵   | 2016-12-10 00:04:33 |
|  2 | 小钱   | 2016-12-10 00:04:48 |
|  3 | 小孙   | 2016-12-13 20:25:00 |
|  4 | 小李   | 2016-12-12 00:00:00 |
+----+--------+---------------------+
{% endhighlight %}

至此，不用再担心被炒鱿鱼了。

常见问题
===
* 有人会问，我DDL误操作了怎么快速回滚？比如drop了一张大表。

 > 很难做到。因为即使在在row模式下，DDL操作也不会把每行数据的变化记录到binlog，所以DDL无法通过binlog回滚。实现DDL回滚，必须要在执行DDL前先备份老数据。确实有人通过修改mysql server源码实现了DDL的快速回滚，我找到阿里的xiaobin lin提交了一个patch。但据我所知，国内很少有互联网公司应用了这个特性。原因的话，我认为最主要还是懒的去折腾，没必要搞这个低频功能，次要原因是会增加一些额外存储。
 > 
 所以，DDL误操作的话一般只能通过备份来恢复。如果公司连备份也不能用了，那真的建议去买张飞机票了。干啥？跑呗

* mysql除了[binlog2sql](https://github.com/danfengcao/binlog2sql)，是否还有其他回滚工具？

	>当然有。阿里彭立勋对mysqlbinlog增加了flashback的特性，这应该是mysql最早有的flashback功能，彭解决的是DML的回滚，并说明了利用binlog进行DML闪回的设计思路。DDL回滚特性也是由阿里团队提出并实现的。这两个功能是有创新精神的，此后出现的闪回工具基本都是对上面两者的模仿。另外，去哪儿开源的Inception是一套MySQL自动化运维工具，这个就比较重了，支持DML回滚，还不是从binlog回滚的，是从备份回滚的，也支持DDL回滚表结构，数据是回滚不了滴~ 还有一种做法叫slave延时备份，搞台不加业务流量的slave，故意延迟一段时间，这其实是在传统办法的基础上去除了实例恢复这步。此法会额外消耗一台机器，我们不推荐这么做。

如有mysql回滚相关的优秀工具优秀文章遗漏，烦请告知。

我的邮箱 danfengcao.info@gmail.com


参考资料
==============
[1] 彭立勋, [MySQL下实现闪回的设计思路](http://www.penglixun.com/tech/database/mysql_flashback_feature.html)

[2] Lixun Peng, [Provide the flashback feature by binlog](https://bugs.mysql.com/bug.php?id=65178)

[3] 丁奇, [MySQL闪回方案讨论及实现](http://dinglin.iteye.com/blog/1539167)

[4] xiaobin lin, [flashback from binlog for MySQL](https://bugs.mysql.com/bug.php?id=65861)

[5] 王竹峰, [去哪儿inception](https://github.com/mysql-inception/inception)

[6] danfengcao, [binlog2sql: Parse MySQL binlog to SQL you want](https://github.com/danfengcao/binlog2sql)

