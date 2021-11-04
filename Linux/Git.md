# Centos8 环境下 Git 安装配置 #
### 注意看我的标题！！！！我这是针对2.33.1版本 ###

## 一、检查本地是否安装 ##
### 1、检查 ###
    rpm -qa | grep maven
### 2、卸载 ###
    rpm -e 已经存在的maven全名	
## 二、解压操作文件 ##
### 下载并上传到 /usr/local/ 文件夹下
#### 下载地址 ####
    https://mirrors.edge.kernel.org/pub/software/scm/git/git-2.33.1.tar.gz
### 解压（找到压缩文件） ###
	tar -zxvf git-2.33.1.tar.gz -C /usr/local/
#### 切换到解压目录下 ####
	cd /usr/local/
#### 查看是否剪切成功 ####
	ls -a
	ll
## 三、安装开发工具 ###
	yum groupinstall -y "Development Tools"
> 安装过程中需要输入’Y’，然后回车等待安装完成
### 四、安装依赖 ###
    yum install -y wget unzip gettext-devel openssl-devel perl-CPAN perl-devel zlib-devel libcurl-devel expat-devel
> 安装过程中需要输入’Y’，然后回车等待安装完成
### 五、构建Git ###
#### 切换到目录下 ####
	cd /usr/local/git-2.33.1
#### 构建 ####
    sudo make all install prefix=/usr/local
#### 查看版本 ####
    git --version
#### 查找Git ####
    whereis git

##### 2、关闭防火墙 #####
	systemctl stop firewalld.service
##### 4、开启防火墙 #####
	systemctl start firewalld.service