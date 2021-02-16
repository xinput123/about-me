## Redis压测
### 1、redis-benchmark -h 127.0.0.1 -p 6379 -c 100 -n 100000
100个并发连接，100000个请求

### 2、redis-benchmark -h 127.0.0.1 -p 6379 -c 100 -q -d 100
存储大小为100字节的数据包

### 3、redis-benchmark -t set, lpush -n 100000 -q
只测试 set 和 lpush

### 4、redis-benchmark -n 100000 -q script load "redis.call('set','foo','bar')"
只测试这条命令