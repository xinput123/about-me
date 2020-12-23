CentOS传送门：https://pkg.jenkins.io/redhat/

在这个界面我们可以看到官网教我们安装的步骤

### 1、拉取Jenkins库
```
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat/jenkins.repo

sudo rpm --import https://pkg.jenkins.io/redhat/jenkins.io.key
```

注意文章所说的2点
1.如果您之前从Jenkins中导入了密钥，那么“rpm—import”将会失败，因为您已经有了一个密钥。请忽略这一点，继续前进。
2.您需要显式地安装Java运行时环境，因为Oracle的Java rpm是不正确的，并且无法注册为提供Java依赖项。因此，在Java中添加显式的依赖项要求将强制安装OpenJDK JVM。

### 2、安装jenkins包,更新
```
yum install -y jenkins

yum update jenkins
```

### 3、jenkins下载完后，有如下目录
```
## jenkins安装目录，WAR包会放在这里
/usr/lib/jenkins/

## jenkins配置文件，“端口”，“JENKINS_HOME”等都可以在这里配置
/etc/sysconfig/jenkins

## 默认的JENKINS_HOME
/var/lib/jenkins/

## Jenkins日志文件
/var/log/jenkins/jenkins.log
```



### 4、查看Jenkins端口
```
cat /etc/sysconfig/jenkins | grep PORT
```
查找JENKINS_PORT，一般为"8080"，一屏显示不下的话，按回车查看，按control+c退出

### 5、开启关闭jenkins服务
```
# 检查Jenkins服务状态
sudo systemctl status jenkins

# 设置为开机自启动
sudo systemctl enable jenkins

# 启动Jenkins服务
sudo systemctl start jenkins

# 关闭jenkins服务
service jenkins stop
```
启动Jenkins时，可能是会出现错误的，因为java的安装路径和Jenkins服务启动默认的java路径可能是不一样的。看到上面的错误提示，找不到或者没有/usr/bin/java这个目录，就可以确定是Jenkins启动服务的java路径还是默认的，并不是我们自己安装jdk是的路径，所以，我们需要把文件/etc/init.d/jenkins中的路径修改过来：我们只需要将
```
candidates="
/etc/alternatives/java
/usr/lib/jvm/java-1.8.0/bin/java
/usr/lib/jvm/jre-1.8.0/bin/java
/usr/lib/jvm/java-1.7.0/bin/java
/usr/lib/jvm/jre-1.7.0/bin/java
/usr/lib/jvm/java-11.0/bin/java
/usr/lib/jvm/jre-11.0/bin/java
/usr/lib/jvm/java-11-openjdk-amd64
/usr/bin/java
```
里面的最后一行修改成我们自己的jdk路径即可。修改完后
```

```

### 6、为Jenkins开启防火墙8080端口：
```
# 检查防火墙配置
sudo firewall-cmd --list-all

# 开启8080端口
sudo firewall-cmd --zone=public --add-port=8080/tcp 
sudo firewall-cmd --zone=public --add-port=8080/tcp --permanent

# 重新加载防火墙配置
sudo firewall-cmd --reload
```