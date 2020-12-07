#### 1、情况：
```
mkdir: /home/logs/java/: Read-only file system
```

#### 2、比较旧的版本上执行
```
sudo mount -uw /
```

#### 3、在新版本中，上述命令会出现错误
```
mount_apfs: volume could not be mounted: Operation not permitted
mount: / failed with 77
```

#### 4、看Google，有以下信息
```
1. Open Terminal and at the prompt type: csrutil status
2. Press return and if it comes back with the status as: System Integrity Protection status: enabled. You'll need to disable it.
```

#### 5、使用 csrutil status 查看 csrutil 的状态，如果是 enable的话，需要修改为 disabled。如何修改，看 6

#### 6、简单来说如下
```
1. Boot to Recovery OS by restarting your machine and holding down the Command and R keys at startup.
2. Launch Terminal from the Utilities menu.
3. Enter the following command: csrutil disable
4. Close Terminal and Restart the computer.
```
- 1、关机重启，安装从实用程序菜单启动终端
- 2、在程序菜单栏打开终端
- 3、输入命令 ： csrutil disable
- 4、关闭终端并重启

#### 7、修改 auto_master 文件
```
sudo vi /etc/auto_master
```
修改（注释/home这一行）
- 修改前：/home auto_home -nobrowse,hidefromfinder
- 修改后：#/home auto_home -nobrowse,hidefromfinder 

#### 8、关闭后 ，查看 csrutil status，如果已经是 disabled，执行
```
sudo mount -uw /
sudo mkdir /home
sudo chmod 777 /home
```
