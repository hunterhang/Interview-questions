# mysql常见问题

### myisam与innodb的区别
*  InnoDB支持事务，MyISAM不支持
*  InnoDB支持外键，而MyISAM不支持
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

### Mysql 锁原理

### Mysql 乐观锁与悲观锁

### Mysql MVCC

### Mysql 同步复制 & 异步复制

### Mysql的最左原则

### Innodb的数据存储结构

### 应用题
* 一个6亿的表a，一个3亿的表b，通过外间tid关联，你如何最快的查询出满足条件的第50000到第50200中的这200条数据记录。
>1、如果A表TID是自增长,并且是连续的,B表的ID为索引select * from a,b where a.tid = b.id and a.tid>500000 limit 200;
2、如果A表的TID不是连续的,那么就需要使用覆盖索引.TID要么是主键,要么是辅助索引,B表ID也需要有索引。
select * from b , (select tid from a limit 50000,200) a where b.id = a .tid;



