### 一、问题描述
在现阶段的开发过程中，使用springcloud，因为每个人负责不同的模块，但是经常一个服务需要调用到n个不同的服务，这时开发团队在会经常遇到这样一个问题，因为大家使用的服务发现都是同一个服务，这个时候就导致了一个服务发现中有很多名字相同的不同实例。但是由于服务开启了自我保护，经常在访问时调用不了，因为其他人已经关闭了这个服务，或者其他问题。

### 二、解决办法
因为本地每个人启动多个服务，会导致电脑吃不消，所以决定使用docker。在每个人电脑上使用一套docker环境，这样每个人的服务都是独立的。建议安装docker for windows。
docker在win10的新版本中已经不需要安装虚拟机了，直接可以运行在Windows的Hyper-V技术上，但是对win10的版本是有要求的，win10版本必须在10586（win10的小版本号）以上而且必须是64位，查看小版本号在命令行中输出cmd就可以看到。如果版本不够，可以通过升级或者下载 win10 易升[win10 易升](https://download.microsoft.com/download/0/4/7/047889D0-578C-4A44-A38F-7F30A6CB3809/current-version/Windows10Upgrade9252.exe)

### 三、我们将服务发现和内部不常用的服务都放到docker中去运行。
##### 3-1、在 C:\Windows\System32\drivers\etc 中修改 hosts 文件。添加
```
10.1.10.129 outsidehost
```
> 10.1.10.129 是本机内网ip。

##### 3-2、在服务发现eureka-server的配置文件application.yml中
```
server:
  port: 9099

eureka:
  instance:
    hostname: outsidehost
  client:
    registerWithEureka: false
    fetchRegistry: false
    service-url:
      defaultZone: http://outsidehost:9099/eureka/

spring:
  application:
    name: eureka-server	
```
>注： eureka.instance.hostname: outsidehost    使用outsidehost的域名。
eureka.client.service.defaultZone: http://outsidehost:9099/eureka/ 注册中心使用域名。

##### 3-3、客户端 cfg-server的配置文件中application.yml修改
```
eureka:
  instance:
    hostname: outsidehost
  client:
    registerWithEureka: true
    fetchRegistry: true
    serviceUrl:
      defaultZone: http://outsidehost:9099/eureka/
```
>注： eureka.instance.hostname: outsidehost    使用outsidehost的域名。
eureka.client.service.defaultZone: http://outsidehost:9099/eureka/ 注册中心使用域名。

##### 3-4、对于其他客户端服务都参照 3-3配置即可，主要注意：
```
eureka.instance.hostname: outsidehost
eureka.client.service.defaultZone: http://outsidehost:9099/eureka/ 
```

##### 3-5、docker中启动
```
docker run -d --add-host=outsidehost:10.1.10.129 -p 9099:9099 --restart=always eureka-server:1.0
docker run --rm -d --add-host=outsidehost:10.1.10.129 -p 9092:9092 wanhuhealth-cfg-server
```
> 如何制作镜像这里就不详细说了。

##### 3-6、编译器中启动其他服务，调用 客户端其他服务。修改application.yml文件。
```
eureka:
  instance:
    hostname: outsidehost
  client:
    service-url:
      defaultZone: http://outsidehost:9099/eureka/ # 服务注册中心地址
```
> 注意事项和 3-3,3-4是一样的。