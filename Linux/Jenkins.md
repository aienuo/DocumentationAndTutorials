# Centos8 环境下 Jenkins 安装配置 #
### 注意看我的标题！！！！我这是针对2.318版本 ###

> Jenkins 安装需要JDK环境，如果没有安装JDK请参考安装 [JDK-1.8](JDK-1.8.md)
> Jenkins 运行需要TomCat，如果没有安装TomCat请参考安装 [TomCat](TomCat.md)

## 一、检查本地是否安装 ##
### 1、检查 ###
    rpm -qa | grep Jenkins
### 2、卸载 ###
    rpm -e 已经存在的 Jenkins 全名	
## 二、解压操作文件 ##
### 下载并上传到 /usr/local/apache-tomcat-8.5.72/webapps 文件夹下
#### 关闭TomCat ####	
	/usr/local/apache-tomcat-8.5.72/bin/shutdown.sh
#### 进入TomCat webapps 文件夹下 ####
    cd /usr/local/apache-tomcat-8.5.72/webapps
#### 删除原来的 ROOT 文件夹 ####
    rm -rf ROOT
#### 下载地址 ####
    wget https://mirrors.tuna.tsinghua.edu.cn/jenkins/war/2.318/jenkins.war
#### 重命名 jenkins 文件 ####
	mv jenkins ROOT
#### 查看是否成功 ####
	ls -a
	ll
## 三、重启验证是否生效 ###
### 启动TomCat 或者 重启计算机 ###
#### 启动TomCat ####
	/usr/local/apache-tomcat-8.5.72/bin/startup.sh
#### 重启计算机 ####
	shutdown -r now
### 查看 TomCat 启动日志 寻找 默认密码
#### 进入TomCat logs 文件夹下 ####
    cd /usr/local/apache-tomcat-8.5.72/logs
#### 最后50行内容 ####
    tail -50 catalina
#### 查看是否出现如下内容 ####
> 如果出现如下内容说明启动成功，
> 其中类似于 0471dca6b053462fb481bf7599b48f3b 这串字符串就是 admin 的默认密码使用此密码进行登录即可
> 
> 在浏览器内访问TomCat 查看是否出现 Jenkins 登录页

    *************************************************************
    *************************************************************
    *************************************************************

    Jenkins initial setup is required. An admin user has been created and a password generated.
    Please use the following password to proceed to installation:
    
    0471dca6b053462fb481bf7599b48f3b
    
    This may also be found at: /root/.jenkins/secrets/initialAdminPassword
    
    *************************************************************
    *************************************************************
    *************************************************************