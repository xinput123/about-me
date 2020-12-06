```
@echo off
echo 开始备份安利数据库
set h=%time:~0,2%
set h=%h: =0%
set bak_filename=anli-%date:~0,4%%date:~5,2%%date:~8,2%-%h%%time:~3,2%%time:~6,2%
echo %bak_filename%
set bakdir=d:\vtradex\bak\
set logdir=d:/vtradex\bak\logs\
exp anli/anli file=%bakdir%%bak_filename%.dmp log=%logdir%%bak_filename%.log
echo 备份完成。。。
echo 正在备份到192.168.3.100\share上
xcopy /y %bakdir%%bak_filename%.dmp \\192.168.3.100\share
echo 备份完成
```
