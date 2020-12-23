访问 [Maven官网](http://maven.apache.org/download.cgi)，找到下载链接
![image.png](https://upload-images.jianshu.io/upload_images/4046640-e106887fe5cc7af3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 1、解压  文件放在/usr/local下
```
tar zxvf apache-maven-3.6.3-bin.tar.gz
```

### 2、添加环境变量
打开 /etc/profile， 添加如下内容：
```
# MAVEN_HOME
MAVEN_HOME=/usr/local/apache-maven-3.5.4
export MAVEN_HOME
export MAVEN_OPTS="-Xms512m -Xmx1024m"
export PATH=${PATH}:${MAVEN_HOME}/bin
```

### 3、执行命令使配置生效
```
source /etc/profile
```

### 4、查看
```
mvn -v
```