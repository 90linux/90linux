# MHA + Maxscale 数据库的高可用和读写分离
1. MySQL 常见发行版本
2. MySQL 标准化、自动化部署
3. 深入浅出MySQL备份与恢复
4. 深入理解MySQL主从复制
5. MySQL构架设计与容量规划
6. MHA
7. Maxscale


## MySQL 常见发行版本
* Mysql 官方
* Percona 
* Mariadb
## MySQL 标准化、自动化部署 
```
1. 机器标准化
2. 参数标准化
3. 统一安装包
4. 目录标准化
5. 多实例部署
6. 自动化部署
```

### 机器标准化
* CPU
* Memory
* SSD
* SAS

### 参数标准化
![](http://i.imgur.com/Sn3VJ2N.png)

### 统一安装包
```
1. 源码包
2. rpm
3. 二进制
```
### 目录标准化

|数据目录|日志目录|binlog|
|:-:|:-:|:-:|
|data|logs|binlog|

### 多实例部署
|实例|u01|u02|u03|u04|
|:-:|:-:|:-:|:-:|:-:|
|端口|3306|3307|3308|3309|
|数据目录|/data/3306/data|/data/3307/data|/data/3308/data|/data/3309/data|
|日志目录|/data/3306/logs|/data/3307/logs|/data/3308/logs|/data/3309/logs|
|硬盘容量|500G+100G|500G+100G|500G+100G|500G+100G|

* 数据目录 ---> SSD 
* 日志目录 ---> SAS 
### 自动化部署
```bash
1. 打通SSH互信(saltstack、Ansible、puppet)
2. 修改主机名(唯一)
3. 创建并挂载目录
4. 安装agent端（salt-minion）
5. 安装zabbix agent端
6. 安装MySQL并加载监控项
7. 调整MySQL 参数（port、server_id、innodb_buffer_pool_size）
8. 启动MySQL 初始化环境（管理用户、查询用户、监控用户）
```
### MySQL 安装步骤
```
关闭防火墙
配置sysctl.conf
检查操作系统上是否安装了MySQL
下载mysql源码包
添加用户和组
配MySQL环境变量
创建目录及授权
解压mysql5.6
MySQL参数配置
初始化MySQL脚本
启动MySQL 
登录MySQL
```

## 深入浅出MySQL备份与恢复
```
备份恢复的使用场景
备份类型
备份有效性测试
自动化备份设计
MySQL备份工具
Xtrabackup安装
Xtrabackup备份实现
innobackupex整个备份过程
innobackupex恢复原理
Innobackupex备份恢复演示
```
### 备份恢复的使用场景
```
监管要求
搭建备库
异常恢复
```
### 备份类型
```
热备
冷备
温备
```
#### 逻辑备份
```
1. 逻辑备份将数据库的内容转储到文本文件中
2. 这些文本文件包含 SQL 语句，这些 SQL 语句包含重建MySQL 数据库和表所需的全部信息
3. 可以使用该文本文件在运行不同体系结构的其他主机上重新装入数据库
4. 在创建逻辑备份时，MySQL 服务器必须处于运行状态，因为服务器在创建文件时要读取备份的表的结构和内容
5. 采用逻辑备份时，可以备份本地和远程的 SQL 服务器。只能在本地 MySQL 服务器上执行其他类型的备份
```
#### 物理备份
```
1. 物理备份是 MySQL 数据库文件的二进制副本。这些副本以完全相同的格式保留数据库存储在磁盘上;
2. 原始备份是数据库文件位的完整表现形式，因此必须将其恢复到使用相同数据库引擎的MySQL 服务器;
3. 在从 InnoDB 表恢复原始 MySQL 备份时，会在目标服务器上保留一个 InnoDB 表;
4. 原始二进制备份的速度比逻辑备份快，因为该过程是简单的文件复制，不需要了解文件的内部结构
```

#### 冷备与热备(物理备份)
冷备(MySQL服务器CLOSE)
```
1. 可通过关闭 MySQL 服务器，然后再进行备份
2. 备份时，必须确保在备份进行期间服务器不修改文件
```
热备(MySQL服务器OPEN)
```
1. 可以使用快照、复制或专有方法，最大限度地减小对 MySQL 和应用程序的影响
2. 对于某些存储引擎，更好的办法是暂时锁定数据库，进行备份，然后再将数据库解锁,锁在热备做了两件事：第一记录binlog文件的位置、第二冷备非事务引擎引的表(MYISAM)
```
### 备份有效性测试
![](http://i.imgur.com/0wNEmWa.png)
### 自动化备份设计
![](http://i.imgur.com/MrebUOL.png)
### MySQL 备份工具
1. mysqldump
```
# mysqldump工具使用
1. mysqldump --help
2.mysqldump -h127.0.0.1 -P3306 -uroot  -p --single-transaction  linux > /tmp/linux.sql
# 分析mysqldump流程
打开general.log————>执行mysqldump————>分析general.log————>关闭general.log
```
2. mysqlduper
```
# 下载
https://launchpad.net/mydumper
# 编译安装
cmake .
make 
make install
# mysqldumper 备份
mydumper  --user=root  --password='123456' --socket=/tmp/mysql3306.sock --regex '^(?!(mysql))'  --outputdir=/u01/backup/ --compress --verbose=3  --logfile=/apps/backup/mydumper.log
```
3. xtrabackup
```
1. Xtrabackup是由percona提供的mysql数据库备份工具，据官方介绍，这也是世界上惟一一款开源的能够对innodb和xtradb数据库进行热备的工。
Xtrabackup中主要包含两个工具：
	1. xtrabackup：是用于热备份innodb, xtradb表中数据的工具，不能备份其他类型的表，也不能备份数据表结构.
	2. innobackupex：是将xtrabackup进行封装的perl脚本，可以备份和恢复MyISAM表以及数据表结构。
```
### Xtrabackup 安装
1. rpm包安装
```shell
rpm -Uvh https://www.percona.com/downloads/XtraBackup/LATEST/percona-xtrabackup-24-2.4.5-1.el6.x86_64.rpm
```
2. yum源安装
```shell
yum install percona-xtrabackup
```
3. 源码安装
```shell
1. 解压源码包
tar -xzvf percona-xtrabackup-2.1.7.tar.gz 
2. 安装perl环境(DBI/DBD)
yum install perl-DBIx-Simple.noarch perl-DBD-MySQL.x86_64  perl*

3. Prerequisites
yum -y install cmake gcc gcc-c++ libaio libaio-devel automake autoconf bison libtool ncurses-devel libgcrypt-devel libev-devel

4. 开始编译
./utils/build.sh	#根据版本确认build.sh的参数
./utils/build.sh innodb56	#开始编译
5.把xtrabackup_5.6复制到/usr/bin下
cp /u01/percona-xtrabackup-2.1.7/src/xtrabackup_56 /usr/bin/
```
### Xtrabackup 备份实现
![](http://i.imgur.com/9WSDc5p.png)

### innobackupex 整个备份过程
![](http://i.imgur.com/o9Xzpll.png)

### innobackupex 恢复原理
![](http://i.imgur.com/3NRNY2c.png)
### innobackupex备份恢复演练
![](http://i.imgur.com/KyBAUvI.png)
## 深入理解MySQL主从复制
```
主从复制的架构
线上快速搭建主从复制
主从复制的详细过程分析
主从复制相关参数
Semi-sync复制
主从复制常见问题
```
### 主从复制的架构
![](http://i.imgur.com/OeCWyBo.png)
### 线上快速搭建主从复制流程
![](http://i.imgur.com/ODSb3lQ.png)
### 线上快速搭建主从复制命令
```sql
Master上创建复制账号并授权
grant replication slave,replication client on *.* to repl@'%' identified by ‘password’;
Master、Slave上分别设置不同的Server_id  vim my.cnf; set global server_id=xxxxx;
Master上执行一次完整逻辑（物理）备份	#ibbackup, xtrabackup, mysqldump  mysqldump	--single-transaction --master-data
Slave上，拿到Master全备在Slave做一次全量恢复
Slave上执行CHANCE MASTER配置主从复制  Slave上执行Start slave启动复制
start slave;
show slave status\G
```
![](http://i.imgur.com/HQ1SR5r.png)
### 主从复制的详细过程分析
![](http://i.imgur.com/2uqo7up.png)
### 主从复制相关参数
```sql
# Master
server-id
read_only  
mysql_log_bin
binlog_format
binlog_cache_size
max_binlog_size  
expire_logs_days  
binlog-do-db  
binlog-ignore-db
#Slave
server-id
read_only
sql_log_bin
log_slave_updates
replicate-do-db  
replicate-ignore-db
replicate-do-table
replicate-ignore-table
```
### Binlog日志格式
* SBR(statement based replicate)
```bash
每一条会修改数据的sql都会记录在binlog中
优点：
不需要记录每一行的变化，减少了
binlog日志量，节约了IO，提高性能。
缺点：
由于记录的只是执行语句，为了这些语  句能在slave上正确运行，因此还必须  记录每条语句在执行的时候的一些相关  信息，以保证所有语句能在slave得到  和在master端执行时候相同 的结果。  另外mysql 的复制,像一些特定函数功  能，slave可与master上要保持一致会  有很多相关问题(如now()函数，  sysdate()，以及uuid()会出现问题).
```
* RBR （row based replicate）
```bash
不记录sql语句上下文相关信息，仅保存哪条记录
被修改。
优点
binlog仅需要记录那一条记录被修改成什么了。
所以row模式的日志内容会非常清楚的记录下每一  行数据修改的细节。而且不会出现某些特定情况  下的存储过程，或function，以及trigger的调用  和触发无法被正确复制的问题.
缺点
所有的执行的语句当记录到日志中的时候，都  将以每行记录的修改来记录，这样可能会产生大  量的日志内容,比如一条update语句，修改多条记  录，则binlog中每一条修改都会有记录，这样造  成binlog日志量会很大.
从数据的安全性考虑出发，推荐使用row模式。
```
* Mixed
```bash
结合SBR与RBR  通常使用SBR  非确定情况使用RBR
是row和statement模式的混合使用，一般的语句
修改使用statment格式保存binlog，如一些函数，  statement无法完成主从复制的操作，则采用row  格式保存binlog,MySQL会根据执行的每一条具体  的sql语句来区分对待记录的日志形式，也就是在  Statement和Row之间选择一种.具体的可以参看  官方的文档:  http://dev.mysql.com/doc/refman/5.6/en/binary-log-mixed.html
```
### Semi-sync复制
>Semi-sync最早是由Google实现的一个补丁，代码主要由Mark Callaghan、Wei Li（@Google）等人  贡献。Google原本是将需求提给Hekki的，但是后来等不及就自己实现了。5.5版本正式合并到官方的版本。
Semi-sync就是保证主库将日志先传输到备库，然后再返回给应用事务提交成功，流程如下:
![](http://i.imgur.com/VQfZXBf.png)

### 主从复制常见问题
* 主库挂了，怎么判断从库是否同步完成？
* mysql主从库同步错误：1032/1062/1060 Error
* 主从的UUID重复的错误  Slave_IO_Running: No  Slave_SQL_Running: Yes
Last_IO_Error: Fatal error: The slave I/O thread stops because master and slave have equal  MySQL server UUIDs; these UUIDs must be different for replication to work.




## MySQL构架设计与容量规划
```
MySQL构架设计
容量评估
Sysbench压测
业务痛点
读写分离方案
数据分区方案
分表的两种方案
混合方案
实现路线分析
业内解决方案
```
### MySQL构架设计
![](http://i.imgur.com/zytJrTp.png)
### 容量评估
![](http://i.imgur.com/19wjwdU.png)
### 容量评估——QPS评估
* 峰值qps=(总的pv * 80%)/(60 * 60 * 24 *20%)
* 机器数=总的峰值qps/压测得出的单台机器极限qps
![](http://i.imgur.com/bMea7fp.png) 
### Sysbench 压测
见sysbench安装及性能测试.docx文档

### 读写分离方案
>海量数据的存储及访问，通过对数据库进行读写分离，来提升数据的处理能力。读写分离它的方案特点是数据库产生多个副本，数据库的写操作都集中到一个数据库上，而一些读的操作呢，可以分解到其它数据库上。这样，只要付出数据复制的成本，就可以使得数据库的处理压力分解到多个数据库上，从而大大提升数据处理能力。
>优点：由于所有的数据库副本，都有数据的全拷贝，因此所有的数据库特性都可以实现，部分机器当机不影响系统的使用。
>缺点：数据的复制同步是一个问题，要么采用数据库自身的复制方案，要么自行实现数据复制方案。需要考虑数据的迟滞性，一致性方面的问题。
### 数据分区方案
>原来所有的数据都是在一个数据库上的，网络IO及文件IO都集中在一个数据库上的，因此CPU、内存、文件IO、网络IO都可能会成为系统瓶颈。而分区的方案就是把某一个或某几张相关的表的数据放在一个独立的数据库上，这样就可以把CPU、内存、文件IO、网络IO分解到多个机器中，从而提升系统处理能力。
优点：不存在数据库副本复制，性能更高。
缺点：分区策略必须经过充分考虑，避免多个分区之间的数据存在关联关系，每个分区都是单点，如果某个分区宕机，就会影响到系统的使用。
### 数据分表方案
>不管是上面的读写分离方案还是数据分区方案，当数据量大到一定程度的时候，都会导致处理性能的不足，这个时候就没有办法了，只能进行分表处理。也就是把数据库当中数据根据按照分库原则分到多个数据表当中，这样，就可以把大表变成多个小表，不同的分表中数据不重复，从而提高处理效率。
优点：数据不存在多个副本，不必进行数据复制，性能更高。
缺点：分表之间的数据很少进行集合运算；分表都是单点，如果某个分表宕机，如果使用的数据不在此分表，不影响使用。
### 分表的两种方案
* 单库单表无法满足大量写的请求
>1. 同库分表：所有的分表都在一个数据库中，由于数据库中表名不能重复，因此需要把数据表名起成不同的名字。
优点：由于都在一个数据库中，公共表，不必进行复制，处理更简单
缺点：由于还在一个数据库中，CPU、内存、文件IO、网络IO等瓶颈还是无法解决，只能降低单表中的数据记录数。表名不一致会导后续的处理复杂。

>2. 不同库分表： 由于分表在不同的数据库中，这个时候就可以使用同样的表名。
优点：CPU、内存、文件IO、网络IO等瓶颈可以得到有效解决，表名相同，处理起来相对简单
缺点：公共表由于在所有的分表都要使用，因此要进行复制、同步。

### 同库分表(分表)
![](http://i.imgur.com/xQMUsqK.png)
### 不同库分表(分库)
![](http://i.imgur.com/0HSXvoV.png)
### 订单分库分表
![](http://i.imgur.com/m7ZqZNK.png)
### 混合方案
>通过上面的描述，我们理解了读写分离，数据分区，数据分表三个解决方案，实际上都各有优点
，也各有缺，因此，实践当中，会把三种方案混合使用。由于数据一天比一天长多，实际上，在  刚开始的时候，可能只采用其中一种方案，随着应用的复杂，  数据量的增长，会逐步采用多个方案混合的方案。以提升处理能力，避免单点。

### 实现线路分析
>正所谓条条大路通罗马，解决这个问题的方案也有多种，但究其深源，都可以归到两种方案之上，一种是对用户透明的方案，即用户只用像普通的J
DBC数据源一样访问即可，由框架解决所有的数据访问问题。另外一种是应用层解决，具体一般是在Dao层进行封装。
JDBC层方案  
优点：开发人员使用非常方便，开发工作量比较小；可以实现数据库无关。  
缺点：框架实现难度比较大，性能不一定能做到最优。  同样是JDBC方案，也有两种解决方案，一种是有代理模式，一种是无代理模式。
有代理模式，有一台专门的代理服务器，来接收用户请求，然后发送请求给数据库集群中的数据，并对数据进行汇集后再提交给请求方。  无代理模式，就是说没有代理服务器，集群框架直接部署在应用访问端。  有代理模式，能够提供的功能更强大，甚至可买提供中间库进行数据处理，无代理模式处理性能较强有代理模式少一次网络访问，相对来说性能更好，但是功能性不如有代理模式。
DAO层方案  
优点：开发人员自由度非常大，性能调优更精准。
缺点：开发人员在一定程度上受影响，与具体的Dao技术实现相关，较难做到数据库无关。  由于需要对SQL脚本进行判断，然后进行路由，因此DAO层优化方案一般都是选用iBatis或Spring Jdbc  Template等方案进行封装，而对于Hibernate等高度封装的OR映射方案，实现起来就非常困难了。
### 分库分表带来的限制
1. 条件查询、分页查询受到限制，查询必须带上分库分表所带上的id
2. 事务可能跨多个库，数据一致性无法通过本地事务实现，无法使用外键
3. 分库分表规则确定以后，扩展变更规则需要迁移数据

### 业务痛点
![](http://i.imgur.com/5wZDwWv.png)、
### 业内解决方案
||来源|血缘|类型|
|:-:|:-:|:-:|:-:|
|Cobar|阿里||中间件|
|TDDL|阿里||客户端二方库|
|DRDS|阿里|Cobar、TDDL|分布式数据库|
|MyCAT|社区|Cobar|中间件|
|Atlas|360|MySQL Proxy|中间件|
|TDSQL|腾讯|MySQL Proxy|分布式数据库|
|Heisenberg|百度|Cobar|中间件|
|蓝海豚|京东|MySQL Proxy|中间件|
|Mxscale|Mariadb||中间件|

## MHA 
### MHA 简介
>MHA，是日本的一位MySQL专家采用Perl语言编写的一个脚本管理工具，  目的在于维持MySQL Replication中Master库的高可用性，其最大特点是可  以修复多个Slave之间的差异日志，最终使所有Slave保持数据一致，然后从  中选择一个充当新的Master，并将其它Slave指向它。

### MHA集群架构
![](http://i.imgur.com/51tKoXG.png)\
### MHA 监控
>每ping_interval秒监控master一次  MHA自身提供了两种监控方式：SELECT和  CONNECT，控制参数ping_type
###
![](http://i.imgur.com/0Zuu8Eq.png)
```bash
•调用SSH脚本对所有Node执行检查，包括
  •Master服务器是否可以SSH连通
  •MySQL实例是否可以连接
  •检查SQL Thread的状态
  •检查哪些Server死掉了，哪些Server是活动  的，以及活动的Slave实例
  •检查Slave实例的配置及复制过滤规则
```
![](http://i.imgur.com/GrIrH0N.png)
```bash
1.SQL Thread alive?	NO: Restart it
2.调用master_ip_failover_script关闭VIP
3.检查各个Slave，获取最近的和最旧的  binary log file和position，并检查各个  Slave成为Master的优先级
4.若dead master所在服务器依然可以通过SSH连通，  则提取dead master的binary log。另外，MHA还要  对各个Slave节点SSH连通性进行检查。
5.调用apply_diff_relay_logs命令恢复Slave的差异日志，  即各个Slave之间的relay log
6.差异日志恢复到New Master上，然后获取New Master的binlog name和
position，最后会开启New Master的写权限
7.清理New Master其实就是重置slave info，即取消原来的Slave信息
```
### MySQL Master Crash
![](http://i.imgur.com/bZeZaAX.png)
### 恢复过程
![](http://i.imgur.com/zAOdXEQ.png)
### Saving binlog events from (crashed) master
![](http://i.imgur.com/JS2XlRo.png)
### Understanding SHOW SLAVE STATUS
![](http://i.imgur.com/ldMEfPb.png)
### Identifying the latest slave
![](http://i.imgur.com/h0OUsxQ.png)
###Identifying what events need to be applied
![](http://i.imgur.com/CD92x38.png)
### MHA部署实战
![](http://i.imgur.com/MJ1UP4c.png)
MHA部署.txt  文档
## Maxscale
![](http://i.imgur.com/1DlCQGe.png)
>maxscale有两种方式实现读写分离。一种是基于connect的，类似haproxy，不解析sql语句。另一种是statement，基于解析sql语句的。解析sql势必会增加性能损耗，我们可以通过php yii框架或者java mybatis框架实现读写分离，基于connect方式，用maxscale做多台slave的负载均衡，从而取代haproxy。如果开发实现困难，那么采用statement方式。
注: maxscale支持主从同步延迟检测功能。
### MariaDB MaxScale - 基于connect
![](http://i.imgur.com/GFAmIDN.png)
### MariaDB MaxScale - 两种模式解释
![](http://i.imgur.com/pU6rQLn.png)
### 用maxscale做多台slave的负载均衡，从而取代haproxy
>大多数公司架构：一个主库，多个从库，主库写，从库负责查询，主库的ha通过MHA实现，从库读的负载均衡通过lvs或者haproxy实现（JAVA框架和PHP框架实现读写分离）
![](http://i.imgur.com/mKxNXfM.png)
### MariaDB MaxScale - 基于SQL解析
![](http://i.imgur.com/FJm4XKt.png)

### maxscale 安装
见 文档