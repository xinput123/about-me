#### 1、查看位置
```
show parameter recover;
```

- DB_RECOVERY_FILE_DEST_SIZE默认是2G

#### 2、查询使用情况语句：
```
select a.SPACE_LIMIT/1024/1024/1024 as "总大小(G)",
a.SPACE_USED/1024/1024/1024 as "使用量(G)",a.NAME as "位置" from  v$recovery_file_dest a;
```

如果开启了归档 可能这个目录增长的比较厉害，如果目录增长到这个设置的大小，数据库就有异常

### 3、可以加大这个参数值 （10g或者100G）
```
alter system set DB_RECOVERY_FILE_DEST_SIZE=10g; 
```

#### 4、如何删除
rman登录 RMAN target 用户名/密码@orcl

```
crosscheck archivelog all;
delete expired archivelog all;

-- 或者删除指定时间之前的archivelog:
DELETE ARCHIVELOG ALL COMPLETED BEFORE 'SYSDATE-7';(指定删除7天前的归档日志)
```
