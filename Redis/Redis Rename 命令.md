## Redis Rename 命令 - 修改 key 的名称
Redis Rename 命令用于修改 key 的名称 。

### 语法
```
RENAME old_Key_Name new_Key_Name
```

### 可用版本
- >= 1.0.0

### 返回值
- 改名成功时提示 OK 
- 失败时候返回一个错误

### 实例
```
# key 存在且 newkey 不存在
 
redis> SET message "hello world"
OK
 
redis> RENAME message greeting
OK
 
redis> EXISTS message               # message 不复存在
(integer) 0
 
redis> EXISTS greeting              # greeting 取而代之
(integer) 1
 
 
# 当 key 不存在时，返回错误
 
redis> RENAME fake_key never_exists
(error) ERR no such key
 
 
# newkey 已存在时， RENAME 会覆盖旧 newkey
 
redis> SET pc "lenovo"
OK
 
redis> SET personal_computer "dell"
OK
 
redis> RENAME pc personal_computer
OK
 
redis> GET pc
(nil)
 
redis:1> GET personal_computer      # 原来的值 dell 被覆盖了
"lenovo"
```