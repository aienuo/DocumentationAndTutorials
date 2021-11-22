# Centos8 环境下 LinuxGameServerManagers 安装配置 #
> LinuxGameServerManagers 安装需要拥有 `SteamCMD`，如果没有安装 `SteamCMD` 请参考 [SteamCMD](SteamCMD.md) 安装
## 一、所需依赖 ##
### 1、EPEL ###
#### 检查 ####
```shell
yum list epel-release
```
#### 安装 ####
```shell
sudo yum install epel-release
```
![EPEL](elel_install.png)
### 2、必备工具 ###
#### 安装 ####
```shell
sudo yum install curl wget tar bzip2 gzip unzip python3 binutils bc jq tmux
```
![必备工具](bibei_install.png)
## 二、安装 ##
### 1、创建用户并登录 ###
#### 创建一个新用户 ####
```shell
useradd arma3server
```
#### 新用户登录 ####
```shell
su - arma3server
```
### 2、下载 ###
> 下载 `linuxgsm.sh` 文件，并赋予可执行权限 `chmod +x linuxgsm.sh` 
```shell
wget -O linuxgsm.sh https://linuxgsm.sh && chmod +x linuxgsm.sh && bash linuxgsm.sh arma3server
```
![LinuxGameServerManagers](linuxgsm_install.png)
### 3、添加 Steam 登录详细信息 ###
#### 创建目录 ####
```shell
mkdir -p /home/arma3server/lgsm/config-lgsm/arma3server
```
#### 配置文件 ####
```shell
vim /home/arma3server/lgsm/config-lgsm/arma3server/common.cfg
```
##### 按一下键盘字母`i`进行编辑 #####
##### 输入以下内容设置 SteamCMD 账号密码 #####
```text
###########################################################################
################################# 默认设置 ################################
###########################################################################
#################### common.cfg - 将设置应用于每个实例 ####################
############################## 游戏服务器设置 #############################
# SteamCMD 登录 | 参考：https://docs.linuxgsm.com/steamcmd#steamcmd-login #
steamuser="username"
steampass='password'
```
##### 按一下`esc`键 退出编辑 #####
##### `:wq` 保存退出 #####
### 4、运行安装程序 ###
```shell
./arma3server install
```
> 执行此步骤 就要耐心等待，网络慢的问题，不要着急
## 三、运维 ##
### 1、启动关闭 ###
#### 启动 ####
```shell
./arma3server start
```
#### 关闭 ####
```shell
./arma3server stop
```
#### 重启 ####
```shell
./arma3server restart
```
#### 控制台 ####
> 控制台允许您查看服务器运行中的实时控制台，并允许您输入命令;如果支持。
```shell
./arma3server console
```
> `CTRL` + `C` 会退出控制台，将终止服务器。
