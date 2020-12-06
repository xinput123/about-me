```
@echo off
set del_days=7
set checkcross_filename=vtradex_rman_checkcross.rman
set delete_batfilename=vtradex_delete_archivelog.bat
set arc_log_file_name=vtradex_del_arc.log
echo ***********************************
echo * *
echo * 正在自动删除%del_days%天前的数据库归档 *
echo * 日志在%arc_log_file_name%中 *
echo * *
echo ***********************************
echo .
echo 正在写命令到文件%checkcross_filename%
echo .
echo crosscheck archivelog all; > %checkcross_filename%
echo DELETE ARCHIVELOG ALL COMPLETED BEFORE 'SYSDATE-%del_days%'; >> %checkcross_filename%
echo exit >> %checkcross_filename%
echo 正在写命令到文件%delete_batfilename%
echo .
echo rman target anli/anli cmdfile=%checkcross_filename% > %delete_batfilename%
echo del /Q %checkcross_filename% >> %delete_batfilename%
echo del /Q %delete_batfilename% >> %delete_batfilename%
echo ------------------------------------------------------->> %arc_log_file_name%
echo 正在执行命令%delete_batfilename%
%delete_batfilename% >> %arc_log_file_name%
exit
```
