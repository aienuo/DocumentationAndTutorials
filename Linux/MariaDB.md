# `Centos8` 环境下 `MariaDB` 的安装配置 #

### 注意看我的标题！！！！我这是针对11.6.2 `[版本](https://downloads.mariadb.org/)`  ###

## 一、检查本地是否安装 ##

### 1、检查 ###

```shell
rpm -qa | grep mariadb
```

### 2、卸载 ###

```shell
rpm -e 已经存在的MariaDB全名
```

## 二、下载 ##

### 1、wget方式下载 ###

```shell
wget https://mirrors.aliyun.com/mariadb///mariadb-11.6.2/bintar-linux-systemd-x86_64/mariadb-11.6.2-linux-systemd-x86_64.tar.gz
```

##### 若报错：`-bash: wget: command not found` 表明没有安装wget，需要执行如下命令安装： #####

```shell
yum -y install wget
```

## 三、解压操作文件 ##

### 1、解压 ###

```shell
tar -zxvf mariadb-11.6.2-linux-systemd-x86_64.tar.gz -C /usr/local/
```

### 2、修改名称 ###

```shell
mv mariadb-11.6.2-linux-systemd-x86_64 /usr/local/mariadb-11.6.2
```

### 3、将文件夹授权给 `mariadb` ###

#### 创建 `mariadb` 组 ####

```shell
groupadd mariadb
```

#### 创建 `mariadb` 用户添加到 `mariadb` 组 ####

```shell
useradd -g mariadb mariadb
```

#### 授权 ####

```shell
chown -R mariadb:mariadb /usr/local/mariadb-11.6.2/
```

### 4、配置 `MariaDB` 环境变量 ###

#### 编辑环境变量文件 ####

```shell
vim /etc/profile
```

#### 按一下键盘字母 `I` 进行编辑，追加如下内容 ####

```
# Set MariaDB Environment
export MARIADB_HOME=/usr/local/mariadb-11.6.2
export PATH=$PATH:${MARIADB_HOME}/bin
```

#### 按一下 `ESC` 键 退出编辑 ####

#### `:wq` 保存退出 ####

#### 执行如下命令即刻生效 ####

```shell
source /etc/profile  
```

## 四、配置启动文件 ###

### 1、修改 `MariaDB` 的 support-files 目录下的 `mysqld_multi.server` 文件 ###

```shell
vim /usr/local/mariadb-11.6.2/support-files/mysqld_multi.server
```

#### 按一下键盘字母 `I` 进行编辑，修改 `mysqld_multi.server` 文件第32-33行 ####

```
basedir=/usr/local/mariadb-11.6.2
bindir=/usr/local/mariadb-11.6.2/bin
```

#### 修改完按一下 `ESC` 键 退出编辑 ####

#### `:wq` 保存退出 ####

### 2、配置 `my.cnf` 文件（启动时自动读取）

```shell
vim /etc/my.cnf
```


