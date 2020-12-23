### 1、官网下载
https://tomcat.apache.org/download-80.cgi

![image.png](https://upload-images.jianshu.io/upload_images/4046640-d3fcd5d13698bdfc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 2、解压以及新建文件
```
tar -zxvf apache-tomcat-8.5.54.tar.gz

mkdir /usr/local/tomcat

mv apache-tomcat-8.5.54 /usr/local/tomcat/
```

### 3、配置tomcat server.xml

### 4、配置防火墙，开放tomcat的端口，如 80
```
firewall-cmd --zone=public --add-port=80/tcp --permanent
 
firewall-cmd --reload
```

### 5、启动 Tomcat
```
/usr/local/tomcat/apache-tomcat-8.5.54/bin/startup.sh
```