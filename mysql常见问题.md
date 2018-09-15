# mysql常见问题

### myisam与innodb的区别
*  InnoDB支持事务，MyISAM不支持;
*  InnoDB支持外键，而MyISAM不支持;
*  InnoDB是聚集索引，MyISAM是非聚集索引; 
> 聚集索引的数据文件是和索引绑在一起的，必须要有主键，通过主键索引效率很高。但是辅助索引需要两次查询，先查询到主键，然后再通过主键查询到数据。因此，主键不应该过大，因为主键太大，其他索引也都会很大。而MyISAM是非聚集索引，数据文件是分离的，索引保存的是数据文件的指针。主键索引和辅助索引是独立的。
* InnoDB不保存表的具体行数，执行select count(*) from table时需要全表扫描。而MyISAM用一个变量保存了整个表的行数，执行上述语句时只需要读出该变量即可，速度很快；
* Innodb不支持全文索引，而MyISAM支持全文索引，查询效率上MyISAM要高；
* 系统奔溃后，MyISAM恢复起来更困难；

### MySQL的复制原理以及流程
> 主：binlog线程——记录下所有改变了数据库数据的语句，放进master上的binlog中；
从：io线程——在使用start slave 之后，负责从master上拉取 binlog 内容，放进 自己的relay log中；
从：sql执行线程——执行relay log中的语句；

### Mysql的主从一致性校验
>主从一致性校验有多种工具 例如checksum、mysqldiff、pt-table-checksum等;

### Mysql 锁
> Mysql锁是为了事务；
* 表锁
> 表锁是由Mysql统一实现的，与存储引擎无关；开销小，并发低；
* 行锁
> 行锁是由InnoDb来实现的；开销大，并发高（MVCC）；
> 
* 锁的模式
> * 共享锁
> * 排它锁
> * 意向锁
> 指获得行锁之前，session必须获得表锁，不需要太关心
> 
* 锁的类型[https://blog.csdn.net/u014698348/article/details/73374044](https://)
>* 行级锁 record locks
>* 区间锁 gap locks
>* 间隙锁next-key locks
>* 插入意向锁
>* 自增锁
### Mysql MVCC[http://www.eu-share.com/blog/show/id/1](https://)
* MVCC解决什么问题？
>MVCC (Multiversion Concurrency Control)，即多版本并发控制技术,它使得大部分支持行锁的事务引擎，不再单纯的使用行锁来进行数据库的并发控制，取而代之的是把数据库的行锁与行的多个版本结合起来，只需要很小的开销,就可以实现非锁定读，从而大大提高数据库系统的并发性能
* MVCC的基本原理
* 当前读与快照读
> 快照读：简单的select 语句；
> 当前读：在事务中的select， update 语句都使用当前读；例如：update操作分为：
> 1) 读； 2）更新
> 
* 死锁的发生[http://www.eu-share.com/blog/show/id/1](https://)


### Mysql 乐观锁与悲观锁，行锁与表锁
* 只有通过索引条件检索数据，InnoDB才使用行级锁，否则，InnoDB将使用表锁！
* for update 只能在begin,commit中使用，申请排它锁才有效；
> 
>假设有A、B两个用户同时各购买一件 id=1 的商品，用户A获取到的库存量为 1000，用户B获取到的库存量也为 1000，用户A完成购买后修改该商品的库存量为 999，用户B完成购买后修改该商品的库存量为 999，此时库存量数据产生了不一致。
```
CREATE TABLE `goods` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(100) DEFAULT NULL,
  `stock` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `idx_name` (`name`) USING HASH
) ENGINE=InnoDB 
```
>
* 悲观锁方案：悲观锁适合写入频繁的场景。
>begin;
select * from goods where id = 1 for update;
update goods set stock = stock - 1 where id = 1;
commit;
>
* 乐观锁方案：乐观锁适合读取频繁的场景。
>#不加锁获取 id=1 的商品对象
```
select * from goods where id = 1
begin;
update goods set stock = stock - 1 where id = 1 and stock = cur_stock;
commit;
```
>
* 带主键或者索引的查询，产生行锁。此时没查询到结果，则不产生锁，如果主键为<>，则产生表锁；
>
```
begin;
select * from goods where id = 1 and name='prod11' for update;
commit;
```
* 不带索引，不带主键的查询产生表锁
```
begin;
select * from goods where name='prod11' for update;
commit;
```
### Mysql 复制
* 异步复制
>异步复制，主库将事务 Binlog 事件写入到 Binlog 文件中，此时主库只会通知一下 Dump 线程发送这些新的 Binlog，然后主库就会继续处理提交操作，而此时不会保证这些 Binlog 传到任何一个从库节点上。
* 全同步复制
>全同步复制，当主库提交事务之后，所有的从库节点必须收到、APPLY并且提交这些事务，然后主库线程才能继续做后续操作。但缺点是，主库完成一个事务的时间会被拉长，性能降低。
* 半同步复制
>半同步复制，是介于全同步复制与全异步复制之间的一种，主库只需要等待至少一个从库节点收到并且 Flush Binlog 到 Relay Log 文件即可，主库不需要等待所有从库给主库反馈。同时，这里只是一个收到的反馈，而不是已经完全完成并且提交的反馈，如此，节省了很多时间。

### InnoDb最左原则

### Innodb的数据存储结构[https://www.kancloud.cn/kancloud/theory-of-mysql-index/41855](https://)

### innodb buffer pool 热点数据
* 基于LRU（least recently used）算法来实现的

### 高可用方案 [https://cloud.tencent.com/developer/article/1031528](https://)

### Mysql的读写分离组件
### 应用题
* 一个6亿的表a，一个3亿的表b，通过外间tid关联，你如何最快的查询出满足条件的第50000到第50200中的这200条数据记录。
>1、如果A表TID是自增长,并且是连续的,B表的ID为索引select * from a,b where a.tid = b.id and a.tid>500000 limit 200;
2、如果A表的TID不是连续的,那么就需要使用覆盖索引.TID要么是主键,要么是辅助索引,B表ID也需要有索引。
select * from b , (select tid from a limit 50000,200) a where b.id = a .tid;



