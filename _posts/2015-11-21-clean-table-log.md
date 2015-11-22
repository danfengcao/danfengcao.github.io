---
layout: post
title:  如何大批量清除MySQL历史数据
date:   2015-11-21 13:14:00
comment: true
category: "MySQL"
---

清除历史数据是DBA经常遇到的一个场景。本文将介绍大批量清除数据的两种常用办法。

---

### 法一：批量、间隔删除

* 用oak-chunk-update工具搞定

    使用oak-chunk-update需要有**UNIQUE KEY**。

    > [oak-chunk-update](https://openarkkit.googlecode.com/svn/trunk/openarkkit/doc/html/oak-chunk-update.html){:target="_blank"}: Perform long, non-blocking UPDATE/DELETE operation in auto managed small chunks.

        oak-chunk-update --user='user' --password='passwd' --host=10.1.1.1 --port=3306  --database=db --execute="DELETE FROM history WHERE clock < unix_timestamp('2015-07-01') and OAK_CHUNK(history)" --chunk-size=30000 --sleep=100 --skip-lock-tables -v


* 亦可自己写脚本，根据主键ID，删除5,000 - 10,000条，sleep 0.1 - 1秒.

#### 注意

* 此法不会立即释放磁盘空间，但新插入数据将填补空缺。空缺填满前，磁盘空间不会再增长；
* 释放空间需要整理碎片(锁表时间会很长)；
* 有些场景下需要立即释放磁盘空间，又不允许长时间锁表，可用法二；


---

### 法二：保留需要数据到新表，删除老表

2.1 建立新表；

    CREATE TABLE new_tbl LIKE tbl;

2.2 在原有表自增ID基础上加上一个值(例如10万)；

    alter table new_tbl AUTO_INCREMENT=增加后的值;

2.3 同时重命名两张表，间接实现数据删除操作；

    RENAME TABLE tbl TO old_tbl, new_tbl TO tbl;

2.4 保留所需数据

* 若数据量不大，可直接插入；

        INSERT INTO new_tbl SELECT * FROM tbl WHERE ID >= ?;

* 若数据量大，须分批缓慢导入到新表。可使用oak-chunk-update或自己写脚本搞定；

2.5 观察一段时间后，删除原表

	DROP TABLE old_tbl;

#### 注意
* 此法可立即释放出磁盘空间，且不会锁表；
* 但在数据完全导回前(步骤4)，会取不到部分历史数据。

