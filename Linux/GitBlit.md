# Centos8 环境下 GitBlit 1.9.1 安装配置 #
### 注意看我的标题！！！！我这是针对1.8版本 ###
## 一、检查本地是否安装 ##
### 1、检查 ###
    rpm -qa | grep Gitblit
### 2、卸载 ###
    rpm -e 已经存在的
## 二、解压操作文件 ##
### 下载并上传到 /usr/local/ 文件夹下
#### 下载地址 ####
    http://gitblit.github.io/gitblit/
### 解压（找到压缩文件） ###
	tar -zxvf gitblit-1.9.1.tar.gz -C /usr/local/
#### 切换到解压目录下 ####
	cd /usr/local/
#### 查看是否剪切成功 ####
	ls -a
	ll
## 三、配置环境变量 ###
#### 1、修改配置文件 ####
	vim /usr/local/gitblit-1.9.1/data/default.properties
##### 按一下键盘字母`i`进行编辑 #####
#### 2、修改以下内容： ####
	server.httpPort = 80
	server.httpsPort = 0
    server.storePassword = ssl加密密码，不能包含“#”
    git.packedGitLimit = 1024m
##### 按一下`esc`键 退出编辑 #####
##### `:wq` 保存退出 #####
#### 3、编辑： ####
	vim /usr/local/gitblit-1.9.1/service-centos.sh
#### 4、修改以下内容： ####
    GITBLIT_PATH=/usr/local/gitblit-1.9.1
    GITBLIT_BASE_FOLDER=/usr/local/gitblit-1.9.1/data
    GITBLIT_HTTP_PORT=81
    GITBLIT_HTTPS_PORT=80
    GITBLIT_LOG=/usr/local/gitblit-1.9.1/log/gitblit.log
    source ${GITBLIT_PATH}/java-proxy-config.sh
    JAVA="java -server -Xmx1024M ${JAVA_PROXY_CONFIG} -Djava.awt.headless=true -cp"
    export JAVA_HOME=/usr/local/jdk1.8.0_311
    export PATH=$PATH:${JAVA_HOME}/bin
##### 按一下`esc`键 退出编辑 #####
##### `:wq` 保存退出 #####
### 四、添加开机自启 ###
#### 执行以下脚本 ####	
	cd /usr/local/gitblit-1.9.1
    ./install-service-centos.sh
### 五、防火墙端口开放 ###
##### 1、查看防火墙状态 #####
	firewall-cmd --state
##### 2、关闭防火墙 #####
	systemctl stop firewalld.service
##### 3、禁止防火墙开机自启 #####
	systemctl disable firewalld.service
##### 4、开启防火墙 #####
	systemctl start firewalld.service
##### 5、Add 添加开放端口 #####
	firewall-cmd --permanent --zone=public --add-port=80/tcp
##### 6、Reload 重新加载 #####
	firewall-cmd --reload
##### 7、检查是否生效 ######
	firewall-cmd --zone=public --query-port=80/tcp
##### 8、重启计算机 #####
	shutdown -r now