# Centos8 环境下 Nginx 安装配置 #
### 注意看我的标题！！！！我这是针对 1.21.3 版本 ###
## 一、检查本地是否安装 ##
### 1、检查 ###
    rpm -qa | grep nginx
### 2、卸载 ###
    rpm -e 已经存在的Nginx全名	
#### 切换到下载目录下 ####
	cd /usr/local/
### 3、下载 ###
	wget https://nginx.org/download/nginx-1.21.3.tar.gz 
### 4、解压（找到压缩文件） ###
	tar -zxvf nginx-1.21.3.tar.gz  -C /usr/local/
#### 切换到解压目录下 ####
	cd /usr/local/nginx-1.21.3
### 5、初始化配置
#### 下载依赖	
	yum -y install make zlib zlib-devel gcc-c++ libtool  openssl openssl-devel
#### 运行./configure 进行初始化配置
	./configure
### 6、执行编译操作
	make
### 7、执行安装操作
	make install
### 8、运行 nginx
#### 进入安装路径
	cd /usr/local/nginx
#### 运行
	./sbin/nginx
#### 查看是否有 Nginx
	ps -ef | grep nginx
### 9、防火墙端口开放 ###
#### 解决 ####
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
##### 8、删除开放端口 ######
    firewall-cmd --zone=public --remove-port=8088/tcp --permanent
### 10、开机自启

### 重启计算机 ###
	shutdown -r now
#### 查看是否有 Nginx
	ps -ef | grep nginx
### 11、维护
#### 进入安装路径
	cd /usr/local/nginx
#### 启动
	./sbin/nginx
#### 重载
	./sbin/nginx -s reload
#### 关闭
	./sbin/nginx -s stop
#### 优雅关闭（当请求被处理完成之后才关闭）
	./sbin/nginx -s quit