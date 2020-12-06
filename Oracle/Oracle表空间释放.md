#### 1、查看表所占空间大小 单位为M
```
SELECT A.segment_name, SUM(A.BYTES)/1024/1024
FROM USER_EXTENTS A 
GROUP BY A.segment_name
ORDER BY SUM(A.BYTES) desc;
```

#### 2、查看ROW_MOVEMENT，默认一般是disable，需要改为 enable
```
SELECT a.ROW_MOVEMENT FROM user_tables a ;
```

#### 3、将 ROW_MOVEMENT 修改为 enable
```
alter table WMS_LOCATION enable row movement;
```

#### 4、释放表空间
```
alter table WMS_LOCATION shrink space
```

#### 5、将 ROW_MOVEMENT 还原为 enable （这一步很重要）
```
alter table WMS_LOCATION disable row movement;
```