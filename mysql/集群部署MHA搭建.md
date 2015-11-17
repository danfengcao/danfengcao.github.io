## 集群部署MHA搭建

### 基本步骤

> 主从搭建-->MHA搭建

#### 主从搭建

> 例，集群名: uodoperation, VIP: 10.1.1.10, 10.1.1.1(M)-->10.1.1.2(S)
库名: UnifiedOrder_Operation

* 无老数据的情况
> 直接主从级联即可。

* 测试环境搭建
> 1. 186恢复至目标机; /home/tools/dp\_db\_recovery/dp\_db\_recovery.pl n /nasbak/mysqlbak/fullbak/db\_20150611*10.1.1.3.gz 10.1.1.1(2) /data/data3306/mysql n;
> 
>  第一个[y/n]:是否需要把线上数据库权限覆盖到目标数据库，y危险；
第二个[y/n]:若目标数据库已存在同名文件，是否覆盖。
> 
> 2. 从库：chang master to ;

* 线上环境搭建
> 1. 186恢复至目标机;
> 2. 从库：chang master to MASTER_HOST=主;
> 3. 主库：chang master to 线上;当前级联为online --> master --> slave;
> 4. 线上全部同步至新集群后，断开online至master级联;

#### MHA搭建

1. 打通ssh key及每个MySQL节点上安装MHA agent 命令行工具包。
> ./mha\_install\_node.sh "10.3.10.56 10.3.10.57" >> /root/caodanfeng/mha\_install\_node.log

2. 找陈云申请VIP，部署MHA
> ./mha\_deploy\_cluster.sh uodoperation 10.1.1.10 "10.1.1.1:3306:1 10.1.1.2:3306:1"
> 
> 1: master候选; 0:切换时，不选为master

3. 在master上配置vip
> /sbin/ifconfig eth0:3306 10.1.1.10/32;/sbin/arping -b -c 2 -w 10 -I eth0 10.1.1.10

4. 查看mha信息
> ./mha\_get\_claster\_info.sh uodoperation








