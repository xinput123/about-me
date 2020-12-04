## 一、配置相关
#### 1、查看mysql启动时加载的配置文件
```
mysql  --help  |  grep  my.cnf
```
![image.png](https://upload-images.jianshu.io/upload_images/4046640-a4b18873564d14bf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
> 这里的文件就是mysql数据库按照顺序读取配置文件的。以最后一个读到的配置为准。

<br/>

## 二、日志文件
#### 1、错误日志 error log
```
show variables like 'log_error';
```
>- 1、当数据库不能正常启动时，第一时间就要看错误日志。

#### 2、慢查询日志 slow query log
默认情况下，Mysql数据库并不启动慢查询日志，需要手动设置。
```
-- 查看是否启用了慢查询日志
show variables like 'slow_query_log';

-- 查看慢查询日志存放的位置
show variables like 'slow_query_log_file';

-- 慢查询时间阈值
show variables like 'long_query_time';

-- 不使用索引的慢查询日志是否记录到索引
show variables like 'log_queries_not_using_indexes';
```
```
-- 服务器运行时间很长，想从慢查询日志中找出执行时间最长的10条SQL语句
mysqldumpslow -s -al -n 10 slow.log
```

#### 3、mysql5.1后可以将慢查询的日志存放到一张表中，slow_log
```
-- 查看慢查询日志输出的位置，默认输出到 FILE
show variables like 'log_output';

-- 设置记录慢查询日志到表中
SET GLOBAL log_output='TABLE';

-- 修改慢日志表引擎，先关闭记录，再修改引擎
SET GLOBAL slow_query_log=off;
ALTER TABLE mysql.slow_log ENGINE=MyISAM;
```

#### 4、二进制文件
二进制文件记录了对Mysql数据库执行更改的所有操作，但是不包括select和show这类操作。只能通过my.cnf去打开二进制文件。
```
-- 先执行一条语句
update t set name='a' where id=100;

-- 查看记录的binlog日志文件对应的file
show MASTER status;

-- 查看binlog事件中是否有记录
SHOW BINLOG EVENTS IN '上一句查询到的file的值';

-- 查看缓冲大小，默认32KB=32768大小
show variables like 'binlog_cache_size';

-- 查看是否使用了临时文件，如果使用临时文件过多，说明需要扩大缓冲大小
show variables like 'binlog_cache%';
```

## 三、show variables like "%timeout%";
![111.jpg](https://github.com/xinput123/about-me/blob/main/MySQL/image/mysql01.jpg)

>- 这个命令
>- **connect_timeout：**连接过程中握手的超时时间，在5.0.52以后默认为10秒，之前版本默认是5秒。
>-**interactive_timeout 和 wait_timeout**的默认值都是28800（8小时）当这两个参数同时出现在里时，会以interactive_timeout的值为准。也就是说不管wait_timeout的值是多少，用show variables like '%timeout%';查看时显示的两个值都是一样的，并且都是interactive_timeout的值。
>- **innodb_lock_wait_timeout 和 innodb_rollback_on_timeout**这个值是针对innodb引擎的，是innodb中行锁的等待超时时间，默认为50秒。如果超时，则当前语句会回滚。如果设置了innodb_rollback_on_timeout，则会回滚整个事务，否则，只回滚事务等待行锁的这个语句。
>- **lock_wait_timeout** 是元数据锁等待超时，任意锁元数据的语句都会用到这个超时参数，默认为一年。元数据锁可以参加[mysql metadata lock](http://dev.mysql.com/doc/refman/5.5/en/metadata-locking.html)，为了保证事务可串行化，不管是myisam还是innodb引擎的表，只要是开始一个事务，就会获取操作表的元数据锁，这时候如果另一个事务要对表的元数据进行修改，则会阻塞直到超时。
>- **net_read_timeout 和 net_write_timeout** 这两个参数在网络条件不好的情况下起作用。比如我在客户端用load data infile的方式导入很大的一个文件到数据库中，然后中途用iptables禁用掉mysql的3306端口，这个时候服务器端该连接状态是reading from net，在等待net_read_timeout后关闭该连接。同理，在程序里面查询一个很大的表时，在查询过程中同样禁用掉端口，制造网络不通的情况，这样该连接状态是writing to net，然后在net_write_timeout后关闭该连接。slave_net_timeout类似。