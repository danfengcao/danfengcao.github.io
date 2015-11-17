## 线上大表DDL

### 基本思想

> 先加从库 --> MHA切换 --> 再加主库

#### 操作前注意事项

1. 从库DDL期间，master binlog是否会被自动清理？
> * master停清binlog，crontab去掉
> * 自动配置推送puppet停掉，pupet配置文件移掉
> 	+ /etc/init.d/puppet stop
> 	+ mv /etc/puppet/puppet.conf /etc/puppet/puppet.conf.tmp

2. 是否有备份工具在跑？会否撑爆磁盘？
> * 关闭备份工具.备份配置相关表：DbInfo.DP_Mng_BackConfig
> * 若已在备份，并且磁盘不足，kill掉。
> * 磁盘空间不足，移除大表后空间并未释放?
> 	- lsof |grep delete 找到对应PID，kill相应程序即可。

3. 磁盘不足，如何加DDL？
> * 换大卡。如何换？
> * 借助临时磁盘。slave停mysql，把表转移到临时磁盘/data01，加软链。
>	- 注意！移动时应该**先copy再删除**，而不是直接mv。
>	- 添加软链接要用全路径。ln -s slink sourcefile

4. zebra有无特殊配置？
> 如active=false，虚实IP配置是否正确

#### 开始操作

一、slave做DDL

1. 再逐个检查一遍注意事项；

2. zebra停slave流量，停主从延迟监控，**别忘记停dw!!!**；关zabbix，slave zabbix not monitor；
> * 每次只能做一个操作，每个操作保存之前必须先测试！
> * **确保已经没流量**。innotop查流量，show processlist查连接情况，/usr/local/dbadmin/bin/kill_process.sh杀连接  
3. 若磁盘空间不够，停mysql，做一个软链，重启mysql；
4. 上DDL，直至结束；**注意：字符集，复制sql时去掉分号**；
5. 加参数**set global slave_type_conversions='ALL_NON_LOSSY';** ！！！
6，start slave；追到数据一致。开zabbix monitor，zebra加slave流量

二、MHA切换

1. 去管理节点10.1.101.50设置MHA参数，ping_interval=1；
2. 重启MHA，/etc/init.d/mha_pctaccountaudit restart；
3. 原master停mysql，/etc/init.d/mysql3306 stop；
4. 查看mha日志，/var/log/mha/mha\_pctaccountaudit/app.log，查看是否切换成功；

5. 查看master丢VIP，slave得VIP，确保切换成功；
> 一主多从，有可能其他从并未指向新主，须检查。
6. 原master启动mysql，指向新的master;
> CHANGE MASTER TO MASTER_HOST='', MASTER_PORT=3306, MASTER_LOG_FILE='', MASTER_LOG_POS=, MASTER_USER='repl', MASTER_PASSWORD='repl';
7. MHA切换完成

三、原master做DDL

 > 重复步骤一

四、重启puppet，binlog定时清理任务。检查zabbix，检查zebra(**勿忘dw**)。


#### 其他知识点

**_slave\_type\_conversions_**

这个参数在mysql5.5.3 引入，目的是启用row格式的bin-log的时候，
如果主从的column的数据类型不一致，会导致复制失败，这个参数的意义就是控制些类型转换容错性。
如果从库的类型比主库类型大，那么复制没有问题的。
如果从库类型比主库类型小，比如从int复制到tinyint这个参数就会起作用。

几种值的设置：

* ALL\_LOSSY： 允许数据截断
* ALL\_NON\_LOSSY： 不允许数据截断，如果从库类型大于主库类型，是可以复制的，翻过来，就不行了，从库报复制错误，复制终止。
* ALL\_LOSSY,ALL\_NON\_LOSSY: 所有允许的转换都会执行，而不管是不是数据丢失。
* 空值： 要求主从库的数据类型必须严格一致，否则都报错。


**出现报错**
> Slave SQL: Column 7 of table 'PCTAccount.TG_UserAccountAudit' cannot be converted from type 'varchar(600)' to type 'int(11)', Error_code: 1677

猜测原因：DDL将字段加到已有字段之间。

方案：在测试环境验证，还原报错。做回滚操作，删除新加字段；再做DDL，将新加字段放到末尾。

