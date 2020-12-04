在回答这个问题之前，我们需要先了解 InnoDB 索引数据结构、数据组织方式。

## 1、存储单元
我们都知道计算机在存储的时候，有最小存储单元。在计算机中磁盘存储的数据最小单元是扇区，一个扇区的大小是512字节，而文件系统(例如 XFS/EXT4) 它的最小单元是块，一个块的大小是4k，而对于我们的InnoDB存储引擎来说，它也有自己的最小存储单元——页(Page)，一个页的大小是16K。

下面几张图可以 帮我们理解最小存储单元：

**文件系统中一个文件大小只有1个字节，但不得不占用磁盘上4KB的空间。**
![一个只有1字节的文件占用磁盘空间数.png](https://github.com/xinput123/about-me/blob/main/MySQL/image/mysql02.png)

**innodb的所有数据文件（后缀为ibd的文件），他的大小始终都是16384（16k）的整数倍。**
![innodb的数据文件.png](https://github.com/xinput123/about-me/blob/main/MySQL/image/mysql03.png)

**磁盘扇区、文件系统、InnoDB存储引擎都有各自的最小存储单元**
![磁盘扇区、文件系统、InnoDB存储引擎都有各自的最小存储单元.png](https://github.com/xinput123/about-me/blob/main/MySQL/image/mysql04.png)

在MySQL中，我们的InnoDB页的大小默认是16k，当然也可以通过参数进行设置：
```
mysql> show variables like 'innodb_page_size';
+------------------+-------+
| Variable_name    | Value |
+------------------+-------+
| innodb_page_size | 16384 |
+------------------+-------+
1 row in set (0.00 sec)
```
**数据表中的数据都是存储在页中，所以一个页中能存储多少行数据呢？**假设一行数据的大小是1k，那么一个页可以存放16行这样的数据。

如果数据库只能按照这样的方式存储，那么如何查找数据就成为了一个问题，因为我们不知道要查找的数据存在哪个页中，也不可能把所有的页都遍历一遍，因为那样太慢了。所以人们想了一个办法，用B+树的方式组织这些数据。如果所示：
![B+树数据.png](https://github.com/xinput123/about-me/blob/main/MySQL/image/mysql05.png)

我们先将数据记录按主键进行排序，分别存放在不同的页中(为了便于理解，这里一个页中只存放3条记录，实际情况下可以存放更多)，除了存放数据的页外，还有存放键值+指针的页，比如图中的 page number = 3 的页，该页存放键值和指向数据页的指针，这样的页由N个键值+指针组成。当然它也是排好序的。这样的数据组织形式，我们称之为**索引组织表**。现在我们来看，查找一条数据，怎么查？

如 select * from user where id = 5 ;

这里的id是主键，我们通过这颗B+树来查找，首先找到根页，你怎么知道user表的根页在哪里呢？其实每张表的根页位置在表空间文件都是固定的，即 page number=3 的页(这点我们下文进一步证明)，找到根页后通过二分查找法，定位的 id = 5 的数据应该在指针P5指向的页中，那么进一步去 page number = 5 的页中查找，同样通过二分查询发即可找到 id = 5 的记录：
|5| zhao2 | 	27 
|:---:|:---:|:---:

###### 现在我们清楚了InnoDB中主键索引B+树是如何组织数据、查询数据的，我们总结一下：
- 1、InnoDB存储引擎的最小存储单元是页，页可以用于存放数据也可以用于存放 键值+指针，在B+树中叶子节点存放数据，非叶子节点存放 键值 + 指针
- 2、索引组织表通过非叶子节点的二分查找法以及指针确定数据在哪个页中，进而在去数据页中查找到需要的数据

那么我们回到开始的问题，通常一颗 B+ 树可以存放多少行数据？
这里我们先假设B+ 树高度为2，及存在一个根节点和若干个叶子节点，那么这颗B+树的存放总记录树为 : 根节点*单个叶子节点记录行数。

上文我们已经说明单个叶子节点（页）中的记录数=16K/1K=16。（这里假设一行记录的数据大小为1k，实际上现在很多互联网业务数据记录大小通常就是1K左右）。

那么现在我们需要计算出非叶子节点能存放多少指针，其实这也很好算，我们假设主键ID为bigint类型，长度为8字节，而指针大小在InnoDB源码中设置为6字节，这样就是14字节，我们一个页中能存放多少这样的单元，其实就代表有多少指针，即 16384 / 14 = 1170。那么可以计算出一颗高度为2的B+树，能存放 1170 * 16 = 18720 条这样的数据记录。

根据同样的原理我们可以算出一个高度为3的B+树可以存放：1170*1170*16=21902400条这样的记录。所以在InnoDB中B+树高度一般为1-3层，它就能满足千万级的数据存储。在查找数据时一次页的查找代表一次IO，所以通过主键索引查询通常只需要1-3次IO操作即可查找到数据。

## 2、怎么得到InnoDB主键索引B+树的高度
上面我们通过推断得出B+树的高度通常是1-3，下面我们从另外一个侧面证明这个结论。在InnoDB的表空间文件中，约定page number为3的代表主键索引的根页，而在根页偏移量为64的地方存放了该B+树的page level。如果page level为1，树高为2，page level为2，则树高为3。即B+树的高度=page level+1；下面我们将从实际环境中尝试找到这个page level。

在实际操作之前，你可以通过InnoDB元数据表确认主键索引根页的page number为3，你也可以从《InnoDB存储引擎》这本书中得到确认。
```
SELECT b.name, a.name, index_id, type, a.space, a.PAGE_NO
FROM information_schema.INNODB_SYS_INDEXES a,
information_schema.INNODB_SYS_TABLES b
WHERE a.table_id = b.table_id AND a.space <> 0;
```
执行结果
![执行结果.jpg](https://github.com/xinput123/about-me/blob/main/MySQL/image/mysql06.png)

可以看出数据库dbt3下的customer表、lineitem表主键索引根页的page number均为3，而其他的二级索引page number为4。

###### 下面我们对数据库表空间文件做想相关的解析：
![数据库表空间文件.png](https://github.com/xinput123/about-me/blob/main/MySQL/image/mysql07.png)

因为主键索引B+树的根页在整个表空间文件中的第3个页开始，所以可以算出它在文件中的偏移量: 16386*3=49152(16384为页的大小)。

另外根据《InnoDB存储引擎》中描述在根页的64偏移量位置前2个字节，保存了page level的值，因此我们想要的page level的值在整个文件中的偏移量为：16384*3+64=49152+64=49216，前2个字节中。

接下来我们用hexdump工具，查看表空间文件指定偏移量上的数据：
![表空间数据.png](https://github.com/xinput123/about-me/blob/main/MySQL/image/mysql08.png)
**linetem表的page level为2，B+树高度为page level+1=3；**
**region表的page level为0，B+树高度为page level+1=1；**
**customer表的page level为2，B+树高度为page level+1=3；**

这三张表的数据量如下：
![统计数据表中数据量.png](https://github.com/xinput123/about-me/blob/main/MySQL/image/mysql09.png)

总结：

lineitem表的数据行数为600多万，B+树高度为3，customer表数据行数只有15万，B+树高度也为3。可以看出尽管数据量差异较大，这两个表树的高度都是3，换句话说这两个表通过索引查询效率并没有太大差异，因为都只需要做3次IO。那么如果有一张表行数是一千万，那么他的B+树高度依旧是3，查询效率仍然不会相差太大。

region表只有5行数据，当然他的B+树高度为1。