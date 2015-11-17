## 大批量DML

### 基本思想

> 批量、间隔操作

#### 一、大量sql语句如何操作？

1. 将文件分割成多个小文件，放到一个目录下
 > split -l 10000 big-update-file.sql > tmp/

2. 对所有小文件做DML操作
 > for i in * ;do echo "mysql -h1.1.1.1 -umyadmin -p'passwd' dbname < $i && sleep 1" && mysql -h1.1.1.1 -umyadmin -p'passwd' dbname < $i && sleep 1;done

#### 二、单sql语句大批量DML如何操作？

如：
> delete from TG_MovieBonusRecord where ID < 1041458

* 法一：将单条sql转换成大量小sql，分批执行

 > for (( i=0; i<1041458; i+=10000)); do let j=i+10000; echo "mysql -h10.1.101.112 -umyadmin -p`cat /root/.myadmin.mp` TGMovie -e 'delete from TG_MovieBonusRecord where ID >= $i and ID < $j' && sleep 1"; done

 > for i in * ;do echo "mysql -h10.1.6.114 -umyadmin -p`cat /root/.myadmin.mp` TPPromoAttribute < $i && sleep 10" && mysql -h10.1.6.114 -umyadmin -p`cat /root/.myadmin.mp` TPPromoAttribute < $i && sleep 10;done

* 法二：若是update或delete，可用工具方便搞定 (**推荐**)

 > [oak-chunk-update](http://code.openark.org/forge/openark-kit): perform long, non-blocking UPDATE/DELETE operation in auto managed small chunks.
 
 > oak-chunk-update --user=myadmin  --password='passwd' --host=10.1.6.195 --port=3306  --database=zabbix --execute="DELETE FROM history WHERE clock < unix_timestamp('2015-07-01') and OAK_CHUNK(history)" --chunk-size=30000 --sleep=100 --skip-lock-tables -v

* 法三：保留需要数据到新表，删除老表 (**不推荐**)

1. 建立新表
 
 > CREATE TABLE new_tbl LIKE tbl;

2. 保留所需数据 
 > 若数据量不大，可直接插入。 
 > 
 > INSERT INTO new_tbl SELECT * FROM tbl WHERE ID >= 1000000; 
 >
 > 若数据量大，须分批缓慢导入到新表。(此步看情况，有些情况会麻烦些)

3. 重命名两张表
 > 同时重命名两张表，间接实现数据删除操作
 >
 > RENAME TABLE tbl TO old_tbl, new_tbl TO tbl; 
 
4. 观察一段时间后，删除原表
  > DROP TABLE old_tbl;

