## Redis PEXPIRE 命令 - 设置 key 的过期时间以毫秒计

Redis PEXPIRE 命令用于设置 key 的过期时间，以毫秒技。key 过期后将不再可用。

### 语法
```
PEXPIRE KEY_NAME TIME_IN_MILLISECONDS
```

### 可用版本
- >= 1.0.0

### 返回值
- 设置成功返回 1 。 
- 当 key 不存在或者不能为 key 设置过期时间时(比如在低于 2.1.3 版本的 Redis 中你尝试更新 key 的过期时间)返回 0 。

### 实例
```
// 首先创建一个 key 并赋值
SET w3ckey redis

// 为 key 设置过期时间
PEXPIRE tutorialspoint 1000
```