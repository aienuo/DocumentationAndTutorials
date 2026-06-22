# Centos8 环境下 X86 rhel7 版本 DM8 的安装配置 #

### 注意看我的标题！！！！我这是针对 X86 的 rhel7 版本离线安装

## 一、检查本地是否安装 ##

### 1、检查 ###

```shell

```

### 2、卸载 ###

```shell

```	

## 二、下载 ##

### 1、官网下载（需要账号登陆） ###

[下载地址](https://download.dameng.com/eco/adapter/DM8/202505/dm8_20250506_x86_rh7_64.zip)

### 2、将下载后的文件上传到 `/usr/local` 文件夹下 ###

* 使用 [windows Terminal 工具](https://apps.microsoft.com/detail/9n8g5rfz9xk3?hl=zh-CN&gl=CN)。 语法： scp 【本地文件地址】 【服务器账号】@【服务器IP地址】:/usr/local/

```shell
scp D:/Downloads/工具安装包/dm8_20250506_x86_rh7_64.zip root@100.110.111.102:/usr/local/
```
### 3、远程登录服务器 ###

* 使用 [windows Terminal 工具](https://apps.microsoft.com/detail/9n8g5rfz9xk3?hl=zh-CN&gl=CN)。 语法： ssh 【服务器账号】@【服务器IP地址】

```shell
ssh root@100.110.111.102
```

## 三、解压操作文件 ##

### 1、解压 ###

```shell
cd /usr/local/
```

```shell
unzip dm8_20250506_x86_rh7_64.zip
```

### 2、仔细阅读 ReadMe 对照本地组件版本是否大于等于要求版本 ###

```shell
cat dm8_20250506_x86_rh7_64.README
```

#### 2.1 查看 `openssl` 版本 ####

```shell
openssl version
```

#### 2.2 查看 `gcc` 版本 ####

```shell
gcc -v
```

#### 2.3 查看 `glibc` 版本 ####

```shell
ldd --version
```

#### 2.4 查看 `glibcxx` 版本 ####

```shell
strings /usr/lib64/libstdc++.so.6 | grep GLIBC
```

##### 没有的组件手动安装，版本低于他的注意升级版本 #####


## 四、升级组件（可选操作） ##

### `openssl` ###

#### 1、下载并上传到 `usr/local/` 文件夹 ####

[openssl-1.1.1w 下载地址](https://artfiles.org/openssl.org/source/openssl-1.1.1w.tar.gz)

### 2、将下载后的文件上传到 `/usr/local` 文件夹下 ###

* 使用 [windows Terminal 工具](https://apps.microsoft.com/detail/9n8g5rfz9xk3?hl=zh-CN&gl=CN)。 语法： scp 【本地文件地址】 【服务器账号】@【服务器IP地址】:/usr/local/

```shell
scp D:/Downloads/工具安装包/openssl-1.1.1w.tar.gz root@100.110.111.102:/usr/local/
```

#### 3、解压 ####

```shell
cd /usr/local/
```

```shell
tar -xzvf openssl-1.1.1w.tar.gz
```

```shell
cd openssl-1.1.1w/
```

#### 4、编译 ####

```shell
sudo ./config --prefix=/usr/local/openssl
```

```shell
make
```

```shell
make install
```

#### 5、备份旧的，给新的创建快捷方式 ####

```shell
mv /usr/bin/openssl /usr/bin/openssl.bak
```

```shell
ln -sf /usr/local/openssl/bin/openssl /usr/bin/openssl
```

```shell
echo “/usr/local/openssl/lib” >> /etc/ld.so.conf
```

#### 6、检查是否成功 ####

```shell
openssl version
```

### `gcc` ###

### `glibc` ###

### `glibcxx` ###


## 五、安装前准备 ##

### 1、创建用户组 及 用户 ###

* 明确指定组ID (GID) 为 2001（而不是由系统自动分配） 

```shell
groupadd dm8db -g 2001
```

* 创建用户 `dm8dba`， 将用户添加到 附加组 `dm8db`， 指定家目录路径为 `/usr/local/dm8db`， 设置用户的 默认登录 Shell 为 Bash， 明确指定用户 ID (UID) 为 2001

```shell
useradd  -G dm8db -m -d /usr/local/dm8db -s /bin/bash -u 2001 dm8dba
```

### 2、修改用户名密码 ###

```Shell
passwd dm8dba
```

### 3、创建文件夹 ###

###### PS：手动在安装路径下新建实例保存目录 `/usr/local/dm8db/data` 、 归档保存目录 `/usr/local/dm8db/arch` 、 备份保存目录 `/usr/local/dm8db/dmbak` 文件夹 ######

```shell
mkdir -p /usr/local/dm8db /usr/local/dm8db/data /usr/local/dm8db/arch /usr/local/dm8db/dmbak
```

### 4、修改目录权限 ###

* 将新建的路径目录权限的用户修改为 `dm8dba`，用户组修改为 `dm8db`

```shell
chown -R dm8dba:dm8db /usr/local/dm8db
chown -R dm8dba:dm8db /usr/local/dm8db/data
chown -R dm8dba:dm8db /usr/local/dm8db/arch
chown -R dm8dba:dm8db /usr/local/dm8db/dmbak
```

*  给路径下的文件设置 `777` 权限

```shell
chmod -R 777 /usr/local/dm8db
chmod -R 777 /usr/local/dm8db/data
chmod -R 777 /usr/local/dm8db/arch
chmod -R 777 /usr/local/dm8db/dmbak
```

* 使用 `root` 用户建立文件夹，待 `dm8dba` 用户建立完成后需将文件所有者更改为 `dm8dba` 用户，否则无法安装到该目录下。

### 5、修改限制参数 ### 

```shell
vim /etc/security/limits.conf
```

##### 按一下键盘字母 `i` 进行编辑 #####

##### 输入以下内容： #####

```conf
dm8dba  soft      nice       0
dm8dba  hard      nice       0
dm8dba  soft      as         unlimited
dm8dba  hard      as         unlimited
dm8dba  soft      fsize      unlimited
dm8dba  hard      fsize      unlimited
dm8dba  soft      nproc      65536
dm8dba  hard      nproc      65536
dm8dba  soft      nofile     65536
dm8dba  hard      nofile     65536
dm8dba  soft      core       unlimited
dm8dba  hard      core       unlimited
dm8dba  soft      data       unlimited
dm8dba  hard      data       unlimited
```

##### 按一下`esc`键 退出编辑 #####

##### `:wq` 保存退出 #####

##### 重启计算机 #####

```shell
shutdown -r now
```

##### 登录系统后切换到 `dm8dba` 账号确认是否生效 ####

```shell
su - dm8dba
```

```shell
ulimit -a
```

### 6、挂载镜像 ###

```Shell
su - root
```

* 输入 `root` 用户密码后 执行脚本

```shell
cd  /usr/local/
```

* 将 `DM8 数据库` 的 `iso` 安装包挂载到 `/mnt`

```shell
mount -o loop dm8_20250506_x86_rh7_64.iso /mnt
```

## 六、数据库安装 ##

### 1、切换到 `dm8dba` 用户并进到 `mnt` 文件夹

```shell
su - dm8dba
```

```shell
cd /mnt
```

* 查看文件是否挂载成功

```shell
ll
```

### 2、执行安装命令进行安装 ###

```shell
./DMInstall.bin -i
```

* Please select the installer's language 输入 `1`

* 是否输入Key文件路径 输入 `n`

* 是否设置时区 输入 `y`

* 请选择时区 输入 `21`

* 请选择安装类型的数字序号 输入 `1`

* 请选择安装目录 此项不输入直接回车

* 是否确认安装路径 输入 `y`

* 略等2~3分钟。 是否确认安装 输入 `y`

### 3、数据库安装完成后，切换到 `root` 用户创建 `DmAPService` ###

```Shell
su - root
```

* 输入 `root` 用户密码后 执行脚本

```Shell
/usr/local/dm8db/dmdbms/script/root/root_installer.sh
```

### 4、 配置环境变量 ###

```shell
cd  /usr/local/dm8db
```

* 配置 `.bash_profile`

```shell
vim .bash_profile
```

##### 按一下键盘字母 `i` 进行编辑 #####

##### 在文本尾部追加以下内容： #####

```conf
export PATH=$PATH:$DM_HOME/bin:$DM_HOME/tool
```

##### 按一下`esc`键 退出编辑 #####

##### `:wq` 保存退出 #####

##### 切换到 `dm8dba` 账号使环境变量生效 ####

```shell
su - dm8dba
```

```shell
source .bash_profile
```

## 七、配置数据库实例 ##

### 1、切换用户 ###

```shell
su - dm8dba
```

```shell
cd /usr/local/dm8db/dmdbms/bin
```

```shell
ll
```

### 2、查看可配置参数 ###

```shell
./dminit help
```

### 3、数据库实例初始化脚本 ###

```shell
./dminit PATH=/usr/local/dm8db/data EXTENT_SIZE=32 PAGE_SIZE=32 CASE_SENSITIVE=y CHARSET=1 SYSDBA_PWD=Dameng08 SYSAUDITOR_PWD=Dameng08 DB_NAME=DAMENG 
INSTANCE_NAME=DMDBSERVER PORT_NUM=6325
```

### 4、注册服务 ###

```Shell
su - root
```

* 输入 `root` 用户密码后 切换目录

```Shell
cd /usr/local/dm8db/dmdbms/script/root
```

```shell
ll
```

*  执行脚本，注册实例服务，数据库名为 DAMENG（对应数据库实例初始化脚本的数据库名 `DAMENG`），-p DMDBSERVER 是服务名的后缀（对应数据库实例初始化脚本的实例名 `DMDBSERVER` ），最终生成的服务名为：DmServiceDMDBSERVER

```shell
./dm_service_installer.sh -t dmserver -dm_ini /usr/local/dm8db/data/DAMENG/dm.ini -p DMDBSERVER
```

* 进入 `/usr/local/dm8db/dmdbms/bin` 目录下，查看目录中生成的服务实例 `DmServiceDMDBSERVER`

```shell
cd /usr/local/dm8db/dmdbms/bin
```

```shell
ll
```

### 5、查看 `DmServiceDMDBSERVER` 服务是否在服务配置中 ###

```shell
sudo systemctl list-dependencies DmServiceDMDBSERVER.service
```

```shell
sudo systemctl status DmServiceDMDBSERVER.service
```

### 6、快捷启动与关闭 ###

##### 启动 #####

```shell
sudo systemctl start DmServiceDMDBSERVER.service
```

##### 关闭 #####

```shell
sudo systemctl stop DmServiceDMDBSERVER.service
```

##### 重启 #####

```shell
sudo systemctl restart DmServiceDMDBSERVER.service
```

##### 查看状态 #####

```shell
sudo systemctl status DmServiceDMDBSERVER.service
```

### 7、重启计算机 ###

```shell
shutdown -r now
```

### 8、验证是否安全启动 ###

```shell
sudo systemctl status DmServiceDMDBSERVER.service
```

## 八、问题解决 ##

### 1、防火墙端口开放，解决远程访问端口不通 ###

##### 查看防火墙状态 #####

```shell
firewall-cmd --state
```

##### 关闭防火墙 #####

```shell
systemctl stop firewalld
```

##### 禁止防火墙开机自启 #####

```shell
systemctl disable firewalld.service
```

##### 开启防火墙 #####

```shell
systemctl start firewalld
```
	
##### Add 添加开放端口 #####

```shell
firewall-cmd --permanent --zone=public --add-port=6325/tcp
```

##### Reload 重新加载 #####

```shell
firewall-cmd --reload
```

##### 检查是否生效 ######

```shell
firewall-cmd --zone=public --query-port=6325/tcp
```

##### 删除开放端口 ######

```shell
firewall-cmd --zone=public --remove-port=6325/tcp --permanent
```


