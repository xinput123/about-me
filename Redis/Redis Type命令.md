## Redis Type 命令 - 返回 key 所储存的值的类型

### 语法
```
TYPE KEY_NAME 
```

### 可用版本
- >= 1.0.0

### 返回值
返回 key 的数据类型，数据类型有：

- none (key不存在)
- string (字符串)
- list (列表)
- set (集合)
- zset (有序集)
- hash (哈希表)

### 实例
```
# 字符串
 
redis> SET weather "sunny"
OK
 
redis> TYPE weather
string
 
 
# 列表
 
redis> LPUSH book_list "programming in scala"
(integer) 1
 
redis> TYPE book_list
list
 
 
# 集合
 
redis> SADD pat "dog"
(integer) 1
 
redis> TYPE pat
set
```