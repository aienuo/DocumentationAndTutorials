# Centos8 环境下 TomCat-8 安装配置 #
### 注意看我的标题！！！！我这是针对8.5.72版本 ###
## 一、检查本地是否安装 ##
### 1、检查 ###
    rpm -qa | grep tomcat
### 2、卸载 ###
    rpm -e 已经存在的TomCat全名	
### 3、下载 ###
	wget https://downloads.apache.org/tomcat/tomcat-8/v8.5.72/bin/apache-tomcat-8.5.72.tar.gz
## 二、解压、操作文件 ##
### 解压（找到压缩文件） ###
	tar -zxvf apache-tomcat-8.5.72.tar.gz -C /usr/local/
#### 切换到解压目录下 ####
	cd /usr/local/
#### 查看是成功 ####
```	
ls -a
```
#### 将当前文件夹授权给 tomcat ####
##### 创建tomcat组 #####
	groupadd tomcat
##### 创建mysql用户添加到mysql组 #####
	useradd -g tomcat tomcat
##### 授权 #####
    chown -R tomcat:tomcat /usr/local/apache-tomcat-8.5.72/
## 三、配置环境变量 ###
#### 1、修改配置文件 ####
	vim /etc/profile
##### 按一下键盘字母`i`进行编辑 #####
#### 2、输入以下内容： ####
	CATALINA_HOME=/usr/local/apache-tomcat-8.5.72
	PATH=$PATH:$CATALINA_HOME/bin
	export CATALINA_HOME
##### 如果有JDK环境变量请使用如下 #####
	JAVA_HOME=/usr/local/jdk1.8.0_201
	JRE_HOME=/usr/local/jdk1.8.0_201/jre
	CATALINA_HOME=/usr/local/apache-tomcat-8.5.72
	CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
	PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin:$CATALINA_HOME/bin
	export JAVA_HOME JRE_HOME CLASS_PATH CATALINA_HOME PATH
##### 按一下`esc`键 退出编辑 #####
##### `:wq` 保存退出 #####
#### 3、修改立即生效： ####
	source /etc/profile
#### 4、配置Tomcat ####
#### 1、修改配置文件 ####
	vim /usr/local/apache-tomcat-8.5.72/conf/server.xml
##### 按一下键盘字母`i`进行编辑 #####
#### 2、修改以下内容： ####
    <Connector port="80" protocol="HTTP/1.1"
               connectionTimeout="20000"
			   URIEncoding="UTF-8"
               redirectPort="8443" />
## 四、查看是否生效 ##
### 启动TomCat ###
	/usr/local/apache-tomcat-8.5.72/bin/startup.sh
### 验证 ###
##### 输入以下指令如果出现一大堆东西说明成功 #####
	ps -ef|grep tomcat
##### 输入以下指令返回页面代码 #####
	curl 127.0.0.1:80
### 关闭TomCat ###	
	/usr/local/apache-tomcat-8.5.72/bin/shutdown.sh
## 五、防火墙端口开放 ##
### 1、查看防火墙状态 ###
	firewall-cmd --state
### 2、关闭防火墙 ###
	systemctl stop firewalld.service
### 3、禁止防火墙开机自启 ###
	systemctl disable firewalld.service
### 4、开启防火墙 ###
	systemctl start firewalld.service
### 5、Add 添加开放端口 ###
	firewall-cmd --permanent --zone=public --add-port=80/tcp
### 6、Reload 重新加载 ###
	firewall-cmd --reload
### 7、检查是否生效 ####
	firewall-cmd --zone=public --query-port=80/tcp
### 8、删除开放端口 ###
    firewall-cmd --zone=public --remove-port=80/tcp --permanent
### 9、重启计算机 ###
	shutdown -r now
## 六、开机自启 ##
### 第一种 ### 
#### 修改 `/etc/rc.d/rc.local` 文件 ####
##### 编辑 #####
	vim /etc/rc.d/rc.local
##### 按一下键盘字母`i`进行编辑 #####
	export JAVA_HOME=/usr/local/jdk1.8.0_191/
	/usr/local/apache-tomcat-8.5.72/bin/startup.sh
##### 按一下`esc`键 退出编辑 #####
##### `:wq` 保存退出 #####
##### 增加脚本的执行权限 #####
    chmod +x /etc/rc.d/rc.local
##### 修改立即生效： #####
	source /etc/rc.d/rc.local


## 七、问题 - Tomcat正常启动，页面访问不到 ##
	 tail -f -n 500 /usr/local/apache-tomcat-7.0.85/logs/catalina.out
	 tail -f -n 500 /data/apache-tomcat-7.0.85/logs/catalina.out
## 八、问题 - Tomcat启动一直卡在信息: `Deploying web application directory /root/sunshine/apache-tomcat-7.0.86/webapps/ROOT` ##
### 查看 `catalina.out` 文件 ###
	tail -f -n 500 /data/apache-tomcat-8.5.72/logs/catalina.out
### 解决 ###
##### 1、找到文件 #####
	cd /usr/local/jdk1.8.0_191/jre/lib/security
##### 2、编辑文件 #####
	vim java.security
###### 第118行附近 ######
	#securerandom.source=file:/dev/random
	securerandom.source=file:/dev/./urandom
##### 3、重启服务 #####
	/usr/local/apache-tomcat-8.5.72/bin/startup.sh
### 原因 ###
> linux或者部分unix系统提供随机数设备是/dev/random 和/dev/urandom，其中urandom安全性没有random高，但random需要时间间隔生成随机数，jdk默认调用random，从而生成随机数时间间隔长从而到时Tomcat启动速度慢