## 一、ab的用法
```
ab  [可选的参数选项] 需要进行压力测试的url
```

|可选参数|描述|详细描述|
|:---:|:---:|:---|
| **-n** | request | 用于指定压力测试总共执行次数 |
| **-c** | concurrency | 用于指定压力测试的并发数 |
| **-t** | timelimit | 等待响应的最大时间(单位：秒) |
| **-s** | timeout |   |
| **-b** | windowsize | TCP发送/接收的缓冲大小(单位：字节) |
| **-B** | address |  |
| **-p** | postfile | 发送POST请求时需要上传的文件，此外还必须设置-T参数 |
| **-u** | putfile | 发送PUT请求时需要上传的文件，此外还必须设置-T参数 |
| **-T** | content-type | 用于设置Content-Type请求头信息，例如：application/x-www-form-urlencoded，默认值为text/plain |
| **-v** | verbosity | 指定打印帮助信息的冗余级别 |
| **-w** |  | 以HTML表格形式打印结果 |
| **-i** |  | 使用HEAD请求代替GET请求 |
| **-x** | attributes | 插入字符串作为table标签的属性 |
| **-y** | attributes | 插入字符串作为tr标签的属性 |
| **-z** | attributes | 插入字符串作为td标签的属性 |
| **-C** | attribute | 添加cookie信息，例如："Apache=1234"(可以重复该参数选项以添加多个) |
| **-H** | attribute | 添加任意的请求头，例如："Accept-Encoding: gzip"，请求头将会添加在现有的多个请求头之后(可以重复该参数选项以添加多个)   |
| **-A**  | attribute | 添加一个基本的网络认证信息，用户名和密码之间用英文冒号隔开  |
| **-P** | attribute | 添加一个基本的代理认证信息，用户名和密码之间用英文冒号隔开  |
| **-X** | proxy:port | 指定使用的代理服务器和端口号，例如:"126.10.10.3:88" |
| **-V** |  | 打印版本号并退出 |
| **-k** |  | 使用HTTP的KeepAlive特性 |
| **-d** |  | 不显示百分比 |
| **-S** |  | 不显示预估和警告信息 |
| **-g** | filename | 输出结果信息到gnuplot格式的文件中 |
| **-e** | filename | 输出结果信息到CSV格式的文件中 |
| **-r** |  | 指定接收到错误信息时不退出程序 |
| **-m** | method | 方法名称 |

## 二、返回数据 ab http://localhost:8080/cities
```
This is ApacheBench, Version 2.3 <$Revision: 1807734 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking localhost (be patient).....done


Server Software:
Server Hostname:        localhost  (服务器主机名)
Server Port:            8080 (服务器端口)

Document Path:          /cities (供测试的URL路径)
Document Length:        24658 bytes (供测试的URL返回的文档大小)

Concurrency Level:      1 (并发数)
Time taken for tests:   0.201 seconds (压力测试消耗的总时间)
Complete requests:      1 (压力测试完成的总次数)
Failed requests:        0 (失败的请求数)
Total transferred:      25131 bytes (传输的总数据量)
HTML transferred:       24658 bytes  (HTML文档的总数据量)
Requests per second:    4.97 [#/sec] (mean)  (平均每秒的请求数)
Time per request:       201.122 [ms] (mean) (所有并发用户(这里是1)都请求一次的平均时间)
Time per request:       201.122 [ms] (mean, across all concurrent requests) (单个用户请求一次的平均时间)
Transfer rate:          122.03 [Kbytes/sec] received (传输速率，单位：KB/s)

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    0   0.0      0       0
Processing:   201  201   0.0    201     201
Waiting:      201  201   0.0    201     201
Total:        201  201   0.0    201     201

Percentage of the requests served within a certain time (ms)
  50%   1362    50%的处理时间在1362ms内
  66%   1679    60%的处理时间在1679ms内
  75%   1909    
  80%   2065    
  90%   2704    90%的处理时间在2704ms内
  95%   3510    
  98%   5350
  99%   8717
 100%  47988 (longest request)
```