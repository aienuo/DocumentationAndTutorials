# Centos8 环境下 Nacos 安装配置 #
### 注意看我的标题！！！！我这是针对2.0.3版本 ###

> Nacos 安装需要JDK环境，如果没有安装JDK请参考安装 [JDK-1.8](Linux/JDK-1.8.md)

## 一、检查本地是否安装 ##
### 1、检查 ###
```shell
rpm -qa | grep Nacos
```
### 2、卸载 ###
```shell
rpm -e 已经存在的Nacos全名
```
## 二、解压操作文件 ##
### 下载并上传到 /usr/local/ 文件夹下
#### 下载地址 ####
```shell
wget https://github.com/alibaba/nacos/releases/alibaba/nacos/releases/download/2.0.3/nacos-server-2.0.3.tar.gz
```
### 解压（找到压缩文件） ###
```shell
tar -zxvf nacos-server-2.0.3.tar.gz -C /usr/local/
```
#### 切换到解压目录下 ####
```shell
cd /usr/local/
```
#### 查看是否剪切成功 ####
```shell
ls -a
```
## 三、数据库配置 ###
> Nacos 需要 MySql 存放用户、租户等配置信息，如果没有安装 MySql 请参考安装 [MySQL](MySQL.md)
### 创建数据库 ### 
```shell
nacos_config
```
### 生成数据库表 ###
> 运行 /usr/local/nacos/conf 下的 nacos-mysql.sql 文件
### 创建用户
```shell
mysql -h 192.168.211.131 -P 3306 -u root -proot 
```
> 根据个人所需 修改 用户名 以及 IP
```shell
CREATE USER `nacos`@`192.168.211.11` IDENTIFIED WITH mysql_native_password BY 'nacos' PASSWORD EXPIRE NEVER;
```
### 设置权限 ###
> 根据个人所需 修改 用户名 以及 IP
```shell
GRANT Alter, Alter Routine, Create, Create Routine, Create Temporary Tables, Create View, Delete, Drop, Event, Execute, Grant Option, Index, Insert, Lock Tables, References, Select, Show View, Trigger, Update ON `nacos\_config`.* TO `nacos`@`192.168.211.11`;
```
## 四、配置环境变量 ###
#### 1、切换到配置文件目录下 ####
```shell
cd /usr/local/nacos/conf
```
#### 2、修改配置文件 ####
```shell
vim application.properties
```
##### 按一下键盘字母`i`进行编辑 #####
#### 2、修改以下内容： ####
##### 19行 访问路径 #####
```shell
server.servlet.contextPath=/nacos
```
##### 21行 访问端口 #####
```shell
server.port=8848
```
##### 33行 数据库类型（只支持MySQL）#####
```shell
spring.datasource.platform=mysql
```
##### 36行 数据源 #####
```shell
db.num=1
```
##### 39行 MySQL URL 只需要修改 IP、Port、数据库名其他不能动 #####
```shell
db.url.0=jdbc:mysql://192.168.211.131:3306/nacos_config?characterEncoding=utf8&connectTimeout=10000&socketTimeout=30000&autoReconnect=true&useUnicode=true&useSSL=false&serverTimezone=UTC
```
##### 40行 数据库帐号 #####
```shell
db.user.0=nacos
```
##### 41行 数据库密码 #####
```shell
db.password.0=nacos
```
##### 按一下`esc`键 退出编辑 #####
##### `:wq` 保存退出 #####
## 五、开机自启 ##
### 配置 startup.sh 文件
```shell
vim /usr/local/nacos/bin/startup.sh
```
##### 按一下键盘字母`i`进行编辑 #####
### 修改 29行 ###
```shell
[ ! -e "$JAVA_HOME/bin/java" ] && JAVA_HOME=/usr/local/jdk1.8.0_201
```
### 30-32行注释掉 ### 
##### 按一下`esc`键 退出编辑 #####
##### `:wq` 保存退出 #####
### 创建配置文件 配置开机自启 ###
```shell
vim /lib/systemd/system/nacos.service
```
##### 按一下键盘字母`i`进行编辑 #####
#### 输入以下内容： ####
> 本机Nacos的文件安装路径，-m standalone表示作为单机启动，不加的话表示集群启动，目前先作为单机启动。
```shell
[Unit]
Description=nacos
After=network.target

[Service]
Type=forking
ExecStart=/usr/local/nacos/bin/startup.sh -m standalone
ExecReload=/usr/local/nacos/bin/shutdown.sh
ExecStop=/usr/local/nacos/bin/shutdown.sh
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```
##### 按一下`esc`键 退出编辑 #####
##### `:wq` 保存退出 #####
#### 设置开机自启
##### 先进行文件生效配置 #####
```shell
systemctl daemon-reload
```
##### 设置为开机启动 #####
```shell
systemctl enable nacos.service
```
##### 启动 Nacos服务 #####
```shell
systemctl start nacos.service
```
##### 启动成功后进行查看 #####
```shell
systemctl status nacos.service
```
##### 关闭 Nacos服务 #####
```shell
systemctl stop nacos.service
```
## 六、防火墙端口开放 ##
### 1、Add 添加开放端口 ###
```shell
firewall-cmd --permanent --zone=public --add-port=8848/tcp
```
### 2、Reload 重新加载 ###
```shell
firewall-cmd --reload
```
### 3、检查是否生效 ####
```shell
firewall-cmd --zone=public --query-port=8848/tcp
```
### 4、重启计算机 ###
```shell
shutdown -r now
```
