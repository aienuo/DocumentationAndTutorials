# Centos8 环境下 Maven 安装配置 #
### 注意看我的标题！！！！我这是针对3.6.3版本 ###

> Maven 安装需要JDK环境，如果没有安装JDK请参考安装 [JDK-1.8](Linux/JDK-1.8.md)

## 一、检查本地是否安装 ##
### 1、检查 ###
    rpm -qa | grep maven
### 2、卸载 ###
    rpm -e 已经存在的maven全名	
## 二、解压操作文件 ##
### 下载并上传到 /usr/local/ 文件夹下
#### 下载地址 ####
    https://mirrors.tuna.tsinghua.edu.cn/apache/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz
### 解压（找到压缩文件） ###
	tar -zxvf apache-maven-3.6.3-bin.tar.gz -C /usr/local/
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
	JAVA_HOME=/usr/local/jdk1.8.0_201
	JRE_HOME=/usr/local/jdk1.8.0_201/jre
	CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
    MAVEN_HOME=/usr/local/apache-maven-3.6.3
	PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin:$MAVEN_HOME/bin
	export JAVA_HOME JRE_HOME CLASS_PATH MAVEN_HOME PATH
##### 按一下`esc`键 退出编辑 #####
##### `:wq` 保存退出 #####
#### 3、修改立即生效： ####
	source /etc/profile
### 四、查看是否生效 ###
#### 查看版本 ####	
	mvn -version
### 五、配置 settings.xml 文件 ###
#### 切换到目录下 ####
	cd /usr/local/apache-maven-3.6.3/conf
#### 配置本地仓库，找到localRepository标签配置 ####
    <localRepository>/usr/local/apache-maven-3.6.3/localRepository</localRepository>
#### 配置maven远程仓库的地址（默认是国外的库，下载jar包时会很慢，所以会配置成国内的远程库 ####
    <mirror>
        <id>aliyunmaven</id>
        <mirrorOf>*</mirrorOf>
        <name>阿里云公共仓库</name>
        <url>https://maven.aliyun.com/repository/public</url>
    </mirror>