# Windows 操作系统下 MySQL数据库实现数据库自动备份的bat文件 #
## 内容如下： ##	
	@echo off
	::mysql_backup.bat
	set hour=%time:~0,2%
	if "%time:~0,1%"==" " set hour=0%time:~1,1%
	set now=%Date:~0,4%%Date:~5,2%%Date:~8,2%%hour%%Time:~3,2%%Time:~6,2%
	::主机ip
	set host=127.0.0.1
	::端口
	set port=3306
	::用户
	set username=root
	::密码
	set password=root
	::要备份数据库的名字，如果有多个库请用空格分隔
	set databasename=property psb
	:: MySQL Bin 目录
	:: 如果在安装配置时添加 MySQL Bin 目录到了环境变量(PATH) ，此处可以留空
	set MYSQL=D:\mysql-5.7.26-winx64\bin\
	::配置SQL备份路径
	set DIR=D:\mysql-5.7.26-winx64\mysql_backup\
	:: 创建备份MySQL的备份目录
	if not exist %DIR% (
		mkdir %DIR% 2>nul
	)

	if not exist %DIR% (
		echo Backup path: %DIR% not exists, create dir failed.
		goto exit
	)

	echo start mysqldump ...

	for %%i in (%databasename%) do (
		%MYSQL%mysqldump -h%host% -P%port% -u%username% -p%password% %%i > %DIR%%%i-%now%.sql 2>nul
	)

	echo end mysqldump
	::记录主库状态，且做日志记录
	%MYSQL%mysql -h%host% -P%port% -u%username% -p%password% -Bse "select now();show master status\G" > %DIR%MySQL_status-%now%.log
	echo delete files before 60 days
	::删除60天前的备份
	forfiles /p "D:\backup\db" /m *.* /d -60 /c "cmd /c del @file /f"

	:exit