## 环境，软件
- centos 7
- docker 1.13.1
- es 5.6.4

## 问题1：权限问题
```
## 创建一个普通用户
useradd es

## 为用户es 创建密码，连续输入两次密码
passwd es

## 创建一个用户组es
groupadd es

## 分配用户es到用户组es中
usermod -G es es

## 进入elasticsearch根目录
cd /url/local/elasticsearch

## 给用户es赋予权限，-R表示逐级（N层目录） ， *表示 任何文件
chown -R es.es *
```

## 问题2
```
max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
```

> 解决方法：在 /etc/sysctl.conf 文件中修改 vm.max_map_count 参数：
```
echo "vm.max_map_count=262144" > /etc/sysctl.conf
sysctl -p
```

## 问题3、ERROR: bootstrap checks failed
```
max file descriptors [4096] for elasticsearch process is too low, increase to at least [65536]
```

> 解决方法： 切换到root用户，编辑 /etc/security/limits.conf 添加类似如下内容
```
* soft nofile 65536
* hard nofile 131072
* soft nproc 2048
* hard nproc 4096
```