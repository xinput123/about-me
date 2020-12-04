### 问题描述
```
2018-08-13 16:50:54  [ qtp2048834776-21:Slf4JLogger:79539 ] - [ ERROR ]  Begin Transaction error !
com.mysql.jdbc.exceptions.jdbc4.CommunicationsException: The last packet successfully received from the server was 40,578,310 milliseconds ago.  The last packet sent successfully to the server was 40,578,315 milliseconds ago. is longer than the server configured value of 'wait_timeout'. You should consider either expiring and/or testing connection validity before use in your application, increasing the server configured values for client timeouts, or using the Connector/J connection property 'autoReconnect=true' to avoid this problem.

	at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
	at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:62)
	at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
	at java.lang.reflect.Constructor.newInstance(Constructor.java:423)
	at com.mysql.jdbc.Util.handleNewInstance(Util.java:425)
```

### 错误原因
Mysql服务器默认的“wait_timeout”是8小时【也就是默认的值默认是28800秒】，也就是说一个connection空闲超过8个小时，Mysql将自动断开该connection，通俗的讲就是一个连接在8小时内没有活动，就会自动断开该连接。而应用连接池却认为该连接还是有效的(因为并未校验连接的有效性)，当应用申请使用该连接时，就会导致上面的报错。

### 解决方法
#### 1、autoReconnect=true
按照日志提示，设置autoReconnect=true会自动连接。但是查看线上配置环境已经这么设置了，网上很多人说mysql客户端5.0后不支持这个配置了，即设置了也没有任何用。

#### 2、dpcp连接池
￼![dpcp属性明细.jpg](https://upload-images.jianshu.io/upload_images/4046640-f27a1474bf4ac3de.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



#### 3、c3p0
通过idleConnectionTestPeriod和maxIdleTime 这两个参数的配置，具体就是必须小于mysq server端的wait_time时间设置
