## 1、前言
[InnoDB的一条记录长什么样子]([https://github.com/xinput123/about-me/blob/main/MySQL/InnoDB%E7%9A%84%E4%B8%80%E6%9D%A1%E8%AE%B0%E5%BD%95%E9%95%BF%E4%BB%80%E4%B9%88%E6%A0%B7%E5%AD%90.md)
) 介绍了 InnoDB 中的一条记录长什么样子，其中提到了记录是存储在「页」中的，那么记录在页中是如何存储的呢？本文进一步介绍 InnoDB 中「页」的相关内容。

## 2、InnoDB数据页
### 2-1、概念&分类
#### 概念
前文提到，InnoDB会把存储的数据划分为若干个「页」，以页作为磁盘和内存交换的基本单位，一个页的默认大小是16KB，查看命令如下(单位是字节)

```
mysql> show status like 'innodb_page_size';
+------------------+-------+
| Variable_name    | Value |
+------------------+-------+
| Innodb_page_size | 16384 |
+------------------+-------+
1 row in set (0.01 sec)
```

#### 分类
InnoDB中的「页」并非只有一种，比如存放 Insert Buffer的页、存放 undo log 的页、存放数据的页等等。其中我们最关注的还是存放我们的表数据的页，又称「索引页」，或者数据页。

### 2-2、数据页结构
#### 2-2-1、数据页结构
InnoDB数据页结构如图所示
![InnoDB 数据页的结构.jpg](https://github.com/xinput123/about-me/blob/main/MySQL/image/mysql14.jpg)

它由七部分构成，简介如下：

| 名称 | 中文名| 占用空间 | 描述 |
|---|---|---|--- 
| File Header | 文件头 | 38字节 | 页的通用信息
| Page Header | 页头部 | 56字节 | 页的专有信息
| Infimum + Supremum | 最小记录和最大记录 | 26字节 | 系统生成的记录，存储页内最大和最小记录
| User Records | 用户记录 | 不确定 | 存储用户记录
| Free Space | 空闲空间 | 不确定 | 页中未使用的空间
| Page Directory | 页目录 | 不确定 | 存储页中记录的位置
| File Trailer | 文件尾 | 8字节 | 校验页是否完整

#### 2-2-2、记录插入过程
在数据页中，当记录为空时，User Records 是不存在的。随着记录的一条条插入，会不断从 Free Space 开辟空间分配给记录，如图所示：
![记录插入过程.jpg](https://github.com/xinput123/about-me/blob/main/MySQL/image/mysql15.jpg)

#### 2.2.3 页结构分析
##### 记录头信息
为了方便描述记录是如何在页中存储的，这里先贴一下前文提到的记录头信息。
![记录头信息.jpg](https://github.com/xinput123/about-me/blob/main/MySQL/image/mysql16.jpg)

##### 它们的各个部分大概说明如下：
- 前两个预留位 ： 暂无用处(各占1位)
- delete_mask : 1位，标记该条记录是否被删除
- min_rec_mask：1位，B+树每层非叶子节点中的最小记录都会添加该标记
- n_owned：4位，当前记录拥有的记录数
- head_no：13位，当前记录在记录堆的位置
- record_type：3 位，记录的类型，记录分为多种类型，使用该位做区分
- - 0 ：普通记录
- - 1：B+树非叶子节点
- - 2：最小记录
- - 3：最大记录
- next_record：16 位，保存下一条记录的相对位置

##### Infimum + Supremum
这部分存储的固定的两条记录，分别是数据页中的「最小记录」和「最大记录」，如果所示：
![Infimum + Supremum.jpg](https://github.com/xinput123/about-me/blob/main/MySQL/image/mysql17.jpg)

**注：由于这两条记录不是我们自定义的，因此不存储在 User Records 空间。**

##### Free Space
空闲空间，数据页中尚未使用的部分

##### User Records
用户记录，这里就是存储我们记录的地方。
上面的「最小记录」和「最大记录」是如何跟我们的记录关联起来的呢？
这时候 next_record 的作用就体现出来了。这两条记录和我们自定义的记录之间是通过 next_record 关联起来的，自定义的记录之间也是通过 next_record 关联起来的，如图所示:
![自定义记录和next_record关系.jpg](https://github.com/xinput123/about-me/blob/main/MySQL/image/mysql18.jpg)

即 「最小记录」的 next_record 指向第一条数据记录，最后一条数据记录的 next_record 指向 「最大记录」。数据记录之间是按照主键的顺序从小到大排序的。
**此处的 (1,100,'aaaa'), (2,200,'bbbb') 等几条内容是自定义的数据。
记录中的蓝色部分为额外信息，橙色部分为记录数据。详细描述可查看前文。**

##### Page Directory
上面是以4条记录距离的，如果要查找其中一条，即时遍历也不会很耗时。但是，如果记录数量一直增加，比如到一条条，一万条甚至更多(加入一个数据页能存的下)，我们如何去查找其中的一条记录呢？这样再去遍历岂不是很笨。

就像我们从一本书里查找某些内容，当只有几页的时候就很容易就能找到；但如果有好几百页，总不能一页一页去翻吧？

我们通常是先查找目录，找到在哪一页，然后直接翻到那一页。页目录的作用跟书的目录差不多。

由于记录之间是按主键排序的，可以把它们从小到大分成一个个的「组」，每组包含很少的几条记录，如图所示：
![Page Directory结构.jpg](https://github.com/xinput123/about-me/blob/main/MySQL/image/mysql19.jpg)

##### 分组规则大致如下：
- 「最小记录」自成一组
- 包含「最大记录」的组一般为1~8条记录
- 其他记录一般是4~8条分为一组

分组之后，把每组中最大的那条记录的地址偏移量提取起来，按顺序存储起来，这些地址偏移量称为槽(Slot)，而这些槽就组成了页目录(Page Directory)。就像给一本书做了目录。

目录有了，怎么使用呢?
**此时如果要查找一条记录，步骤大致如下：**
- 1、由于页目录中的槽是有序的，因此可以使用「二分法」快速定位到一个槽
- 2、找到该槽所在分组中主键值最小的记录
- 3、通过next_record 遍历数组中的记录

##### Page Header
页头部，主要记录数据页的一些信息，比如本页存了多少条记录、页目录中有多少个槽等等。

##### File Header
页头部存的是一个数据页的概要信息，是一个页专有的，而文件头存的是各种页的通用信息，比如页的类型是什么、页的编号是多少，上一页的页号是多少、下一页的页号是多少等等。

既然有上一页、下一页的定义，说明页与页之间其实是相互连接的，它们之间就像一个双向链表那样，如图所示：
![数据页的关联.jpg](https://github.com/xinput123/about-me/blob/main/MySQL/image/mysql20.jpg)

##### File Trailer
文件尾，主要用于校验一个页是不是完整的。

## 3、总结
![InnoDB 数据页的结构.jpg](https://github.com/xinput123/about-me/blob/main/MySQL/image/mysql21.jpg)

