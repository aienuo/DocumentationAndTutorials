# Contos7环境下TomCat-7安装配置 #
### 注意看我的标题！！！！我这是针对7.0.96版本 ###
## 一、检查本地是否安装 ##
### 1、检查 ###
    rpm -qa | grep tomcat
### 2、卸载 ###
    rpm -e 已经存在的TomCat全名	
### 3、下载 ###
	wget http://www-eu.apache.org/dist/tomcat/tomcat-7/v7.0.96/bin/apache-tomcat-7.0.96.tar.gz 
## 二、解压、操作文件 ##
### 解压（找到压缩文件） ###
	tar -zxvf apache-tomcat-7.0.96.tar.gz -C /usr/local/
#### 切换到解压目录下 ####
	cd /usr/local/
#### 查看是否剪切成功 ####
<del>mv /usr/local/apache-tomcat-7.0.96 /data</del>
	ls -a
	ll
#### 将当前文件夹授权给 tomcat ####
##### 创建mysql组 #####
	groupadd tomcat
##### 创建mysql用户添加到mysql组 #####
	useradd -g tomcat tomcat
##### 授权 #####
    chown -R tomcat:tomcat /usr/local/apache-tomcat-7.0.96/
<del>chown -R root:root /data/apache-tomcat-7.0.96/</del>
## 三、配置环境变量 ###
#### 1、修改配置文件 ####
	vim /etc/profile
##### 按一下键盘字母`i`进行编辑 #####
#### 2、输入以下内容： ####
	CATALINA_HOME=/usr/local/apache-tomcat-7.0.96
	PATH=$PATH:$CATALINA_HOME/bin
	export CATALINA_HOME
##### 如果有JDK环境变量请使用如下 #####
	JAVA_HOME=/usr/local/jdk1.8.0_191
	JRE_HOME=/usr/local/jdk1.8.0_191/jre
	CATALINA_HOME=/usr/local/apache-tomcat-7.0.96
	CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
	PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin:$CATALINA_HOME/bin
	export JAVA_HOME JRE_HOME CLASS_PATH CATALINA_HOME PATH
##### 按一下`esc`键 退出编辑 #####
##### `:wq` 保存退出 #####
#### 3、修改立即生效： ####
	source /etc/profile
## 四、查看是否生效 ##
### 启动TomCat ###
	/usr/local/apache-tomcat-7.0.96/bin/startup.sh
### 关闭TomCat ###	
	/usr/local/apache-tomcat-7.0.96/bin/shutdown.sh
### 验证 ###
##### 重启计算机 #####
	shutdown -r now
##### 输入以下指令如果出现一大堆东西说明成功 #####
	ps -ef|grep tomcat
##### 输入以下指令返回页面代码 #####
	curl 127.0.0.1:8080
<del> 
## 五、设置开机自启 ##
### 修改 `/etc/rc.d/rc.local` 文件 ###
#### 编辑 ####
	vim /etc/rc.d/rc.local
#### 添加 ####
##### 按一下键盘字母`i`进行编辑 #####
	export JAVA_HOME=/usr/local/jdk1.8.0_191/
	/usr/local/apache-tomcat-7.0.96/bin/startup.sh
##### 按一下`esc`键 退出编辑 #####
##### `:wq` 保存退出 #####
##### 修改立即生效： #####
	source /etc/rc.d/rc.local
</del>

## 六、防火墙端口开放 ##
### 1、查看防火墙状态 ###
	firewall-cmd --state
### 2、关闭防火墙 ###
	systemctl stop firewalld.service
### 3、禁止防火墙开机自启 ###
	systemctl disable firewalld.service
### 4、开启防火墙 ###
	systemctl start firewalld.service
### 5、Add 添加开放端口 ###
	firewall-cmd --permanent --zone=public --add-port=8080/tcp
### 6、Reload 重新加载 ###
	firewall-cmd --reload
### 7、检查是否生效 ####
	firewall-cmd --zone=public --query-port=8080/tcp