### 环境
centos 7，生产环境建议Jenkins单独部署，内存大一点
Git、Tomcat、Jenkins

### Jdk安装
[jdk安装](https://github.com/xinput123/about-me/blob/main/%E7%8E%AF%E5%A2%83%E5%AE%89%E8%A3%85/Linux/jdk%E5%AE%89%E8%A3%85.md)

### Maven 安装
[Maven 安装](https://github.com/xinput123/about-me/blob/main/%E7%8E%AF%E5%A2%83%E5%AE%89%E8%A3%85/Linux/Maven%E5%AE%89%E8%A3%85.md)

### Tomcat 安装
[Tomcat 安装](https://github.com/xinput123/about-me/blob/main/%E7%8E%AF%E5%A2%83%E5%AE%89%E8%A3%85/Linux/Tomcat%E5%AE%89%E8%A3%85.md)

### Jenkins安装
[Jenkins安装](https://github.com/xinput123/about-me/blob/main/%E7%8E%AF%E5%A2%83%E5%AE%89%E8%A3%85/Linux/Jenkins%E5%AE%89%E8%A3%85.md)

### 安装插件
- Email Extension Plugin (邮件通知)
- GIT plugin (可能已经默认安装了)
- Publish Over SSH (远程Shell)

### 1、配置Jenkins基本信息
首页 ==》系统管理 ==》系统设置
![image.png](https://upload-images.jianshu.io/upload_images/4046640-7555931b97f5ea90.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 2、配置邮件
![image.png](https://upload-images.jianshu.io/upload_images/4046640-eb3dd12bc1403b0d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 3、配置Jdk
Mange Jenkins ==》Global Tool Configuration
![image.png](https://upload-images.jianshu.io/upload_images/4046640-d4333db0fa3d7e70.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
不推荐自动安装的，因为需要oracle的账号密码

### 4、配置Maven
Mange Jenkins ==》Global Tool Configuration
路径为maven的setting.xml路径
![image.png](https://upload-images.jianshu.io/upload_images/4046640-edc686a6c67f9731.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 5、配置Git
Mange Jenkins ==》Global Tool Configuration
![image.png](https://upload-images.jianshu.io/upload_images/4046640-9845357e8ab90e34.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 6、配置Publish over SSH
远程执行shell脚本 采用公钥私钥连接 其中Key里贴的是私钥 远程被管理的主机里贴的是公钥,这2台主机就是相互信任，这样scp等操作就不需要输入用户名和密码。

公钥私钥生成方法：
- 1、管理主机linux 上 ssh-keygen -t rsa -C "mousycoder@foxmail.com 一路回车 会在/root/.ssh下生成id_rsa(私钥) id_rsa.pub(公钥)。
- 2、copy 公钥的内容到远程需要通信(被管理)的主机 /root/.ssh/authorized_keys 如无此目录文件则手动创建。
![image.png](https://upload-images.jianshu.io/upload_images/4046640-f27a1a1f153a07c6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 7、创建job，里面有一段shell脚本
```
#!/bin/bash -ile
OLD_BUILD_ID=$BUILD_ID
echo $OLD_BUILD_ID
BUILD_ID=DONTKILLME
#执行脚本
cd $WORKSPACE
cd doctor/deploy
sh -e build-docs.sh
#返回scp
cd ../

scp -r *.zip 192.168.2.2:/apps/product/temp/doctor/backend/
rm -fr *.zip
ssh 192.168.2.2 /root/shell/doctor_backend.sh
BUILD_ID=$OLD_BUILD_ID
echo $BUILD_ID
```

```
#!/bin/bash -ile
OLD_BUILD_ID=$BUILD_ID
echo $OLD_BUILD_ID
BUILD_ID=DONTKILLME
#执行脚本
cd $WORKSPACE
sh -e build.sh
#返回scp
scp target/*.zip 192.168.2.2:/apps/product/temp/tasks
rm -fr *.zip
ssh 192.168.2.2 /root/shell/tasks.sh
BUILD_ID=$OLD_BUILD_ID
echo $BUILD_ID
```

```
#!/bin/bash -ile
cd $WORKSPACE
rm -fr build
rm -fr build.zip
SASS_BINARY_SITE=https://npm.taobao.org/mirrors/node-sass/
sh -e build.sh
zip -r build.zip build
scp -r build.zip 192.168.2.2:/apps/product/temp/doctor/frontend/
ssh 192.168.2.2 /root/shell/doctor_frontend.sh
```