# Contos7环境下JDK1.8安装配置 #
### 注意看我的标题！！！！我这是针对1.8版本 ###
## 一、检查本地是否安装 ##
### 1、检查 ###
    rpm -qa | grep jdk
### 2、卸载 ###
    rpm -e 已经存在的JDK全名	
## 二、解压操作文件 ##
### 解压（找到压缩文件） ###
	tar -zxvf jdk-8u191-linux-x64.tar.gz -C /usr/local/
#### 切换到解压目录下 ####
	cd /usr/local/
#### 查看是否剪切成功 ####
	ls -a
	ll
## 三、配置环境变量 ###
#### 1、修改配置文件 ####
	vim /etc/profile
##### 按一下键盘字母`i`进行编辑 #####
#### 2、输入以下内容： ####
	JAVA_HOME=/usr/local/jdk1.8.0_191
	JRE_HOME=/usr/local/jdk1.8.0_191/jre
	CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
	PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
	export JAVA_HOME JRE_HOME CLASS_PATH PATH
##### 按一下`esc`键 退出编辑 #####
##### `:wq` 保存退出 #####
#### 3、修改立即生效： ####
	source /etc/profile
### 四、查看是否生效 ###
#### 查看版本 ####	
	java -version
#### 编译器 ####	
	javac