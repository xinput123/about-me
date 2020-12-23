使用yum安装

### 1、更新yum
```
yum update
```

### 2、查找系统已安装的jdk组件
```
rpm -qa | grep java

rpm -qa | grep jdk
```
显示
![image.png](https://upload-images.jianshu.io/upload_images/4046640-24c5de0a40c9c7b2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 3、查看java版本
```
java -version
```

### 4、卸载以前已有的jdk
```
rpm -e --nodeps java-1.7.0-openjdk-headless-1.7.0.261-2.6.22.2.el7_8.x86_64
```

### 5、下载并解压jdk
去oracle官网下载,并解压
```
tar zxvf jdk-8u251-linux-x64.tar.gz
```
### 6、添加到环境变量.
编辑/etc/profile文件

```
#jdk
export JAVA_HOME=/usr/local/jdk1.8.0_251
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
```

### 7、执行命令使配置生效
```
source /etc/profile
```