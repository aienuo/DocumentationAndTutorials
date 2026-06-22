# Centos8 环境下 MySQL Community Server 8.4.5 LTS 的安装配置 #

### 注意看我的标题！！！！我这是针对 [MySQL Community Server 8.4.5 LTS 版本](https://cdn.mysql.com//Downloads/MySQL-8.4/mysql-8.4.5-linux-glibc2.28-x86_64.tar.xz "MySQL Community Server 8.4.5 LTS 版本")

[版本查询地址一](https://dev.mysql.com/downloads/mysql/)

[版本查询地址二](https://downloads.mysql.com/archives/community/)

## 一、检查本地是否安装 ##

### 1、检查 ###

```shell
rpm -qa | grep mysql
```

### 2、卸载 ###

```shell
rpm -e 已经存在的MySQL全名
```	

## 二、下载 ##

### 1、wget方式下载 ###

```shell
wget https://cdn.mysql.com//Downloads/MySQL-8.4/mysql-8.4.5-linux-glibc2.28-x86_64.tar.xz
```

##### 若报错：`-bash: wget: command not found` 表明没有安装wget，需要执行如下命令安装： #####

```shell
yum -y install wget
```

## 三、解压操作文件 ##

### 1、解压 ###

```shell
tar -xvf mysql-8.4.5-linux-glibc2.28-x86_64.tar.xz -C /usr/local/
```

### 2、创建mysql组 ###

```shell
groupadd mysql
```

### 3、创建mysql用户添加到mysql组 ###

```shell
useradd -g mysql mysql
```

## 四、配置启动文件 ###

### 1、切换到 `support-files` 目录下 ###

```shell
cd /usr/local/mysql-8.4.5-linux-glibc2.28-x86_64/support-files/
```

### 2、修改 `mysqld_multi.server` 文件第17-18行数据库安装路径 ###

```shell
vim /usr/local/mysql-8.4.5-linux-glibc2.28-x86_64/support-files/mysqld_multi.server
```

##### 按一下键盘字母 `i` 进行编辑 #####

##### 输入以下内容： #####

```shell
basedir=/usr/local/mysql-8.4.5-linux-glibc2.28-x86_64
bindir=/usr/local/mysql-8.4.5-linux-glibc2.28-x86_64/bin
```

##### 按一下`esc`键 退出编辑 #####

##### `:wq` 保存退出 #####

### 3、修改 `mysql.server` 文件第46-47行、第66-67行、第70行、第72-73行数据库安装路径 ###

```shell
vim /usr/local/mysql-8.4.5-linux-glibc2.28-x86_64/support-files/mysql.server
```

##### 按一下键盘字母 `i` 进行编辑 #####

##### 修改以下内容： #####

```shell
basedir=/usr/local/mysql-8.4.5-linux-glibc2.28-x86_64
```

```shell
datadir=/usr/local/mysql-8.4.5-linux-glibc2.28-x86_64/data
```

```shell
bindir=/usr/local/mysql-8.4.5-linux-glibc2.28-x86_64/bin
```

```shell
sbindir=/usr/local/mysql-8.4.5-linux-glibc2.28-x86_64/bin
```

```shell
libexecdir=/usr/local/mysql-8.4.5-linux-glibc2.28-x86_64/bin
```

##### 按一下`esc`键 退出编辑 #####

##### `:wq` 保存退出 #####

### 4、配置 `my.cnf` 文件 ###

###### PS：修改配置文件前手动在安装路径下新建 `data` 、 `logs` 文件夹，在其下分别创建两个名为 `binlog` 、 `errlog` 的文件夹 ######

```shell
mkdir -p /usr/local/mysql-8.4.5-linux-glibc2.28-x86_64/data /usr/local/mysql-8.4.5-linux-glibc2.28-x86_64/logs /usr/local/mysql-8.4.5-linux-glibc2.28-x86_64/logs/binlog /usr/local/mysql-8.4.5-linux-glibc2.28-x86_64/logs/errlog
```

##### ①、创建 `my.cnf` 文件 #####

```shell
touch /usr/local/mysql-8.4.5-linux-glibc2.28-x86_64/my.cnf
```

##### ②、编辑 `my.cnf` 文件 #####

```shell
vim /usr/local/mysql-8.4.5-linux-glibc2.28-x86_64/my.cnf
```

##### 参考：（注意修改：`basedir` 、 `datadir` 、 `server-id` 、 `log-bin` 、 `log-error` 、 `binlog-do-db`） #####

##### 按一下键盘字母 `i` 进行编辑 #####

##### 输入以下内容： #####

```ini
## 数据库基本配置 ##

[mysql]
# 添加默认字符集参数
default-character-set = UTF8MB4
# 修改MYSQL端口号，默认为3306，建议不要用默认的，请配置为其他端口号，例如：3369、6033等
port = 6033

[client]
# 添加默认字符集参数
default-character-set = UTF8MB4
# 修改MYSQL端口号，默认为3306，建议不要用默认的，请配置为其他端口号，例如：3369、6033等
port = 6033

[mysqld]
# 设置端口
port = 6033
# 设置mysql的安装目录
basedir = /usr/local/mysql-8.4.5-linux-glibc2.28-x86_64
# 设置mysql数据库的数据的存放目录	 #8.0以下版本需要配置数据目录；MySQL 8+ 不需要以下配置，系统自己生成即可，否则有可能报错
datadir = /usr/local/mysql-8.4.5-linux-glibc2.28-x86_64/data
# 允许最大连接数
#max_connections = 200
# 配置允许连接失败的次数。这是为了防止有人从该主机试图攻击数据库系统
max_connect_errors = 10
# 配置mysql在关闭一个非交互的连接之前所要等待的秒数，其取值范围为1-2147483(Windows)，1-31536000(linux)，默认值28800。
#wait_timeout = 31536000
# 配置mysql在关闭一个交互的连接之前所要等待的秒数(交互连接如mysql gui tool中的连接)，其取值范围随wait_timeout变动，默认值28800。
#interactive_timeout = 31536000
# 服务端使用的字符集默认为8比特编码的latin1字符集
character-set-server = UTF8MB4
# 字符集的校对规则
collation-server = utf8mb4_general_ci
# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB
# 创建模式
sql_mode = NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES
# 此配置是在8.4以下的版本中的配置方法，8.4无此项
#default_authentication_plugin = mysql_native_password
# MySQL8.4版本及以上添加，启用验证密码插件（非8.4及以上无需添加）
mysql_native_password=ON
# 添加不区分表/字段大小写项
lower_case_table_names = 1

## 数据库主从搭建配置 ##

# 服务器标志号，注意在配置文件中不能出现多个这样的标识，如果出现多个的话mysql以第一个为准，一组主从中此标识号不能重复。
server-id = 201
# 开启bin-log，并指定文件目录和文件名前缀。
log-bin = /usr/local/mysql-8.4.5-linux-glibc2.28-x86_64/logs/binlog/bin-log
# 错误日志存放路径
log-error = /usr/local/mysql-8.4.5-linux-glibc2.28-x86_64/logs/errlog/master-error.log
# 每个bin-log最大大小，当此大小等于500M时会自动生成一个新的日志文件。一条记录不会写在2个日志文件中，所以有时日志文件会超过此大小。
max_binlog_size = 500M
# 日志缓存大小
binlog_cache_size = 128K
# 需要同步的数据库名字，如果是多个，就以此格式在写一行即可。
#binlog-do-db = database_test
# 不需要同步的数据库名字，如果是多个，就以此格式在写一行即可。（binlog-do-db,binlog-ignore-db 为互斥关系，只需设置其中一项即可）
binlog-ignore-db = mysql,information_schema,performance_schema,sys
# 当Slave从Master数据库读取日志时更新新写入日志中，如果只启动log-bin 而没有启动log-slave-updates则Slave（从）只记录针对自己数据库操作的更新。
log-slave-updates
# 设置bin-log日志文件格式为：MIXED，可以防止主键重复。
binlog_format = "MIXED"
# 解除bin-log限制存储函数的创建、修改、调用
#log_bin_trust_function_creators = 1
# 时区
default-time-zone = '+8:00'
```

##### 按一下`esc`键 退出编辑 #####

##### `:wq` 保存退出 #####

### 5、将 `my.cnf` 文件和 `mysql-8.4.5-linux-glibc2.28-x86_64` 文件夹授权给 `mysql` 用户 ###

```shell
chown mysql:mysql /usr/local/mysql-8.4.5-linux-glibc2.28-x86_64/my.cnf
```

```shell
chmod 750 /usr/local/mysql-8.4.5-linux-glibc2.28-x86_64/my.cnf
```

```shell
chown -R mysql:mysql /usr/local/mysql-8.4.5-linux-glibc2.28-x86_64/
```

### 6、复制 my.cnf 到 /etc/my.cnf（MySQL启动时自动读取）

```shell
cp /usr/local/mysql-8.4.5-linux-glibc2.28-x86_64/my.cnf /etc/my.cnf
```

##### 注意： 如果你在安装时Linux虚拟机时同时安装了默认的mysql，此时操作以上步骤，终端将会提示你文件已存在是否覆盖，输入 `yes` 覆盖即可。 #####

```shell
cp: overwrite ‘/etc/my.cnf’? yes
```


## 五、初始化 ##

### 1、进入bin目录 ###

```shell
cd /usr/local/mysql-8.4.5-linux-glibc2.28-x86_64/bin/
```

### 2、初始化数据库 ###

#### ①、执行数据库初始化脚本 ####

```shell
./mysqld --defaults-file=/usr/local/mysql-8.4.5-linux-glibc2.28-x86_64/my.cnf --basedir=/usr/local/mysql-8.4.5-linux-glibc2.28-x86_64/ --datadir=/usr/local/mysql-8.4.5-linux-glibc2.28-x86_64/data/ --user=mysql --lower-case-table-names=1 --initialize --console
```

#### ②、查询数据库初始化后的 `密码` 留作后用 #### 

```shell
sudo grep 'temporary password' /usr/local/mysql-8.4.5-linux-glibc2.28-x86_64/logs/errlog/master-error.log
```

### 3、安全启动数据库 ###

> 注意执行此操作后 当前窗口冻结了，不可再进行其他操作！，需要重新开启一个黑窗口

```shell
./mysqld_safe
```

### 4、登录数据库 ###

#### ①、本地登录账号 #### 

> 进入 `bin` 目录

```shell
cd /usr/local/mysql-8.4.5-linux-glibc2.28-x86_64/bin/
```

> 当 MySQL 服务已经运行时, 我们可以通过 MySQL 自带的客户端工具登录到 MySQL 数据库中, 首先打开命令提示符, 输入以下格式的命名:

```shell
./mysql -h [host] -P [port] -u [username] -p [password]	
```

- [host] : 指定客户端所要登录的 MySQL 主机名, 登录本机(localhost 或 127.0.0.1)该参数可以省略;
- [port] : 指定客户端所要登录的数据库端口号，默认为 3306 可省略;
- [username] : 登录的用户名;
- [password] : 告诉服务器将会使用一个密码来登录, 如果所要登录的用户名密码为空, 可以忽略此选项。

> 按回车确认, 如果安装正确且 MySQL 正在运行, 会得到以下响应:

```shell
Enter password:
```

> 输入前面记录的 `密码` 进入数据库控制台

#### ②、重置 `root` 密码 ####

```sql
ALTER USER 'root'@'localhost' IDENTIFIED BY '新密码';
FLUSH PRIVILEGES;
```

#### ③、创建远程账号 ####

```sql
CREATE USER 'root'@'%' IDENTIFIED BY '新密码';
FLUSH PRIVILEGES;
```

#### ④、授予数据库权限（操作三选一） ####

> A、授予特定数据库的只读权限

```sql
GRANT SELECT ON 数据库名.* TO '数据库用户'@'%';
FLUSH PRIVILEGES;
```

> B、授予特定数据库的所有权限

```sql
GRANT ALL PRIVILEGES ON 数据库名.* TO '数据库用户'@'%';
FLUSH PRIVILEGES;
```

> C、授予所有数据库的所有权限

```sql
GRANT ALL PRIVILEGES ON *.* TO '数据库用户'@'%' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```

### 5、退出登录 ###

> 修改完成 退出 `quit`。 再次进入时，就可以使用用户名root和刚才设置的新密码登录。

```shell
quit;
```

## 六、开机服务启动设置 ##

### 1、服务启动后，直接运行 `mysql -u root -p` 即可登录，不需要进入到对应的目录 ###

```shell
ln -s /usr/local/mysql-8.4.5-linux-glibc2.28-x86_64/bin/mysql /usr/bin
```

### 2、把 `support-files/mysql.server` 拷贝为 `/etc/init.d/mysql-8.4.5-linux-glibc2.28-x86_64` ###

```shell
cp -a /usr/local/mysql-8.4.5-linux-glibc2.28-x86_64/support-files/mysql.server /etc/init.d/mysql-8.4.5-linux-glibc2.28-x86_64
```

### 3、查看 `mysql-8.4.5-linux-glibc2.28-x86_64` 服务是否在服务配置中 ###

```shell
chkconfig --list mysql-8.4.5-linux-glibc2.28-x86_64
```

### 4、若没有注意看提示语，手动把把 `mysql-8.4.5-linux-glibc2.28-x86_64` 注册为开机启动的服务

```shell
chkconfig --add mysql-8.4.5-linux-glibc2.28-x86_64
```

### 5、查看 `mysql-8.4.5-linux-glibc2.28-x86_64` 服务是否配置成功 ###

```shell
chkconfig --list mysql-8.4.5-linux-glibc2.28-x86_64
```

##### 成功如下： #####
```shell
Note: This output shows SysV services only and does not include native
	systemd services. SysV configuration data might be overridden by native
	systemd configuration.

	If you want to list systemd services use 'systemctl list-unit-files'.
	To see services enabled on particular target use
	'systemctl list-dependencies [target]'.

mysql           0:off   1:off   2:on    3:on    4:on    5:on    6:off
```

> PS：若在Ubuntu中,则需要安排软件支持 `chkconfig` 安装 #####

```shell
sudo apt install sysv-rc-conf
sudo cp /usr/sbin/sysv-rc-conf /usr/sbin/chkconfig
```

### 6、快捷启动与关闭 ###

##### 启动 #####

```shell
service mysql-8.4.5-linux-glibc2.28-x86_64 start
```

##### 关闭 #####

```shell
service mysql-8.4.5-linux-glibc2.28-x86_64 stop
```

##### 重启 #####

```shell
service mysql-8.4.5-linux-glibc2.28-x86_64 restart
```

##### 查看状态 #####

```shell
service mysql-8.4.5-linux-glibc2.28-x86_64 status
```

### 7、重启计算机 ###

```shell
shutdown -r now
```

### 8、验证是否安全启动 ###

> 检索

```shell
ps -ef | grep mysql
```

> 查看状态

```shell
service mysql-8.4.5-linux-glibc2.28-x86_64 status
```

## 七、登录 ##

### 1、进入bin目录 ###

```shell
cd /usr/local/mysql-8.4.5-linux-glibc2.28-x86_64/bin/
```

### 2、登录 ###

```shell
./mysql -h [host] -P [port] -u [username] -p [password]	
```

### 3、操作 ### 

> 可进行各类数据库操作

> 部分敏感权限操作需要刷新才会生效

```sql
flush privileges;
```

### 4、退出登录 ###

> `quit` 或者 `exit`


## 八、问题解决 ##

### 1、防火墙端口开放，解决远程访问端口不通 ###

##### 查看防火墙状态 #####

```shell
firewall-cmd --state
```

##### 关闭防火墙 #####

```shell
systemctl stop firewalld
```

##### 禁止防火墙开机自启 #####

```shell
systemctl disable firewalld.service
```

##### 开启防火墙 #####

```shell
systemctl start firewalld
```
	
##### Add 添加开放端口 #####

```shell
firewall-cmd --permanent --zone=public --add-port=3306/tcp
```

##### Reload 重新加载 #####

```shell
firewall-cmd --reload
```

##### 检查是否生效 ######

```shell
firewall-cmd --zone=public --query-port=3306/tcp
```

##### 删除开放端口 ######

```shell
firewall-cmd --zone=public --remove-port=8088/tcp --permanent
```

### 2、`temporary failure in name resolution` 问题，或者是服务正常启动，访问效率特别慢（适用于MySQL\MyCat\TomCat） ###

#### 解决（DNS的问题） ####

##### ①、查看主机名称 #####

	hostname

##### ②、配置 `vim /etc/hosts`	#####

	127.0.0.1   主机名称 localhost localhost.localdomain localhost4 localhost4.localdomain4
	::1         localhost localhost.localdomain localhost6 localhost6.localdomain6