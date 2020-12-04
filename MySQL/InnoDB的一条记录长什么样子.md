## 1、前言
其中存储引擎包括多种，它们存储数据的形式也不尽相同。有的存储引擎（比如 Memory 引擎）只把数据存储在内存、并不会持久化到磁盘，这样一旦服务器挂了，数据就没了。而默认的 InnoDB 引擎则会把数据持久化到磁盘中。

那么，我们插入的一条记录在 InnoDB 引擎中以什么格式存储的呢？本文进一步介绍和分析。

## 2、InnoDB 记录格式
### 2-1、页
InnoDB将存储的数据划分为若干个「页」，以页作为磁盘和内存交互的基本单位，一个页的大小默认为16KB([为什么页的大小默认为16KB呢]())。可以通过下面命令查看默认页的大小(单位是字节)

```
mysql> show status like 'innodb_page_size';
+------------------+-------+
| Variable_name    | Value |
+------------------+-------+
| Innodb_page_size | 16384 |
+------------------+-------+
1 row in set (0.01 sec)
```
也就是说，即便我们只查询一条记录，InnoDB也会至少把16KB的内容从磁盘读取到内存中。

### 2-2、记录格式
InnoDB 的行格式有四种，分别是 Compact、Redundant、Dynamic 和 Compressend，它们在原理上大体都是相同的。本文主要分析 Compact 格式。

可以使用下面的命令查看默认行格式(此处MySQL版本为5.7)：

```
mysql> show variables like 'innodb_default_row_format';
+---------------------------+---------+
| Variable_name             | Value   |
+---------------------------+---------+
| innodb_default_row_format | dynamic |
+---------------------------+---------+
1 row in set (0.00 sec)

# 或者
mysql> SELECT @@innodb_default_row_format;
+-----------------------------+
| @@innodb_default_row_format |
+-----------------------------+
| dynamic                     |
+-----------------------------+
1 row in set (0.00 sec)
```
也可以在建表时指定行格式，或者建表后再修改。

**Compact 行格式的结构大概如图所示：**
![Compact 行格式的结构.jpg](https://github.com/xinput123/about-me/blob/main/MySQL/image/mysql10.jpg)
主要分为两部分：记录的额外信息和记录的真实数据。下面分别介绍。

### 2-3、额外信息
主要分为两部分：记录的额外信息和记录的真实数据。下面分别介绍。

```
mysql> show create table t1\G;
*************************** 1. row ***************************
       Table: t1
Create Table: CREATE TABLE `t1` (
  `c1` varchar(10) DEFAULT NULL,
  `c2` varchar(10) NOT NULL,
  `c3` char(10) DEFAULT NULL,
  `c4` varchar(10) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=latin1
1 row in set (0.00 sec)
```

#### 2-3-1、变长字段长度列表
###### 概念说明
MySQL中有些数据类型的字段长度是不固定的，比如VARCHAR(M)类型、TEXT等，这就导致每条记录中该字段的 「实际」长度可能是不一样的。

为此，MySQL在存储这些变长类型的数据时，实际上分成了两部分存储，分别是：
- 1、真实的数据
- 2、数据占用的字节数

其中数据占用的字节数就保存着在「变长字段长度列表」中。它是以列的「逆序」存储表中变长子弹的实际长度的。

###### 举例分析
以上面的t1表为例，它的变长字段为 c1、c2、c4，若有一条数据如下：

```
mysql> select * from t1;
+------+----+------+------+
| c1   | c2 | c3   | c4   |
+------+----+------+------+
| a    | bb | ccc  | dddd |
+------+----+------+------+
1 row in set (0.00 sec)
```
则这三个字段「c1、c2、c4」占用的的字节数分别是：1、2、4，以「逆序」存放在变长字段长度列表为 040201 (十六进制)。

**注：字节数跟字符集有关系，latin1 字符默认占用一个字节。**

#### 2-3-2、NULL值列表
###### 概念说明
MySQL中有些列是允许为NULL的，如果这些列特别多、每个NULL值都在表中存储的话会很占用空间。Compact把这些NULL统一起来管理，放到了NULL值列表。

它的处理过程如果：
**- 1、统计表中允许为NULL的列**
**- - 1.1、若列都不允许为NULL，则NULL值列表就不存在了**
**- - 1.2、否则、以一个「二进制」来表示一个允许为空的列，仍是「逆序」排列，其中1表示NULL，0表示非NULL**
**- 2、若NULL值列表不足整数「字节」，在高位补0**

###### 举例分析
以上面 t1 表为例，c1、c3、c4 三列都允许为 NULL，则使用 3 位表示三个允许为空的列，不足一个字节（8 位），因此高 5 位补 0，如图所示：

![NULL值列表.jpg](https://github.com/xinput123/about-me/blob/main/MySQL/image/mysql11.jpg)

再插入一条数据（'a', 'bb', NULL, NULL）：

```
mysql> select * from t1;
+------+----+------+------+
| c1   | c2 | c3   | c4   |
+------+----+------+------+
| a    | bb | ccc  | dddd |
| a    | bb | NULL | NULL |
+------+----+------+------+
2 rows in set (0.00 sec)
```
由于第一条记录的字段都不是NULL值，因此它的NULL值列表为: 0000 0000；
第二条记录的 c3、c4 列为NULL，它的NULL值列表为 0000 0110.

#### 2-3-2、记录头信息
第三部分是记录头信息(个人觉得有点类似JVM中的对象头信息)，该部分占用5个字节，示意图如下：
![记录头信息.jpg](https://github.com/xinput123/about-me/blob/main/MySQL/image/mysql12.jpg)

###### 其中各个部分大概说明如下：
- 前两个预留位 ： 暂无用处(各占1位)
- delete_mask : 1位，标记该条记录是否被删除
- min_rec_mask：1位，B+树每层非叶子节点中的最小记录都会添加该标记
- n_owned：4位，当前记录拥有的记录数
- head_no：13位，当前记录在记录堆的位置
- record_type：3 位，记录的类型，记录分为多种类型，使用该位做区分
- next_record：16 位，保存下一条记录的相对位置

### 2-4、真实数据
真实数据部分大体分为两个部分：影藏列 和 自定义数据

#### 2-4-1、影藏列
这部分是InnoDB默认添加的列。主要包括三部分：

##### row_id
真实名称是 DB_ROW_ID，表示行记录的唯一标识，这一列并不是必须的 。row_id在这里不细说了，如果有需要的可以自定查找一下。

##### transaction_id & roll_pointer
transaction_id 的真实名称为 DB_TRX_ID ，表示事务的id；roll_pointer 真实名称为 DB_ROLL_PTR，表示回滚指针。这两者都是跟事务密切相关，而且是必须的，后面再进行分析。

#### 2-4-2、我们自定的数据列
这部分才是我们真正的自定义的列的数据，以 t1 表为例，就是我们定义的 c1、c2、c3、c4 这四列的数据，不再赘述。

## 3、小结，整理图如下
![ InnoDB 引擎的 Compact 行格式.jpg](https://github.com/xinput123/about-me/blob/main/MySQL/image/mysql13.jpg)










