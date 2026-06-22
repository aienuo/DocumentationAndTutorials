# Centos8 环境下 X86 版本 PostgreSQL-18.3 的安装配置 #

### 注意看我的标题！！！！我这是针对 X86 的 PostgreSQL-18.3 版本离线安装

## 一、检查本地是否安装 ##

### 1、检查 ###

```shell

```

### 2、卸载 ###

```shell

```	

## 二、下载 ##

### 1、官网下载（需要账号登陆） ###

[PostgreSQL-18.3 下载地址](https://ftp.postgresql.org/pub/source/v18.3/postgresql-18.3.tar.gz)

### 2、将下载后的文件上传到 `/usr/local` 文件夹下 ###

* 使用 [windows Terminal 工具](https://apps.microsoft.com/detail/9n8g5rfz9xk3?hl=zh-CN&gl=CN)。 语法： scp 【本地文件地址】 【服务器账号】@【服务器IP地址】:/usr/local/

```shell
scp D:/Downloads/工具安装包/postgresql-18.3.tar.gz root@100.110.111.106:/usr/local/
```

## 三、解压操作文件 ##

### 1、远程登录服务器 ###

* 使用 [windows Terminal 工具](https://apps.microsoft.com/detail/9n8g5rfz9xk3?hl=zh-CN&gl=CN)。 语法： ssh 【服务器账号】@【服务器IP地址】

```shell
ssh root@100.110.111.106
```

### 2、进入目录解压 `tar.gz` ###

```shell
cd /usr/local/
```

```shell
tar -zxvf postgresql-18.3.tar.gz  -C /usr/local/
```

## 四、安装前准备 ##

### 1、创建文件夹 ###

* 手动在安装路径下新建实例保存目录 `/usr/local/postgresql/data` 、 日志保存目录 `/usr/local/postgresql/logs` 、 存储与 PostgreSQL 相关的其他文件或配置 `/usr/local/postgresql/conf` 文件夹

```shell
mkdir -p /usr/local/postgresql /usr/local/postgresql/data /usr/local/postgresql/logs /usr/local/postgresql/conf
```

* 检查是否成功

```shell
ls -ld /usr/local/postgresql /usr/local/postgresql/data /usr/local/postgresql/logs /usr/local/postgresql/conf
```

### 2、创建用户组 及 用户 ###

* 明确指定组ID (GID) 为 2001（而不是由系统自动分配） 

```shell
groupadd postgresqldb -g 2001
```

* 创建用户 `postgresqldba`， 将用户添加到 附加组 `postgresqldb`， 指定家目录路径为 `/usr/local/postgresql`， 设置用户的 默认登录 Shell 为 Bash， 明确指定用户 ID (UID) 为 2001

```shell
useradd -g postgresqldb -m -d /usr/local/postgresql -s /bin/bash -u 2001 postgresqldba
```

* 修改用户名密码

```Shell
passwd postgresqldba
```

### 4、准备依赖包（关键步骤） ###

* 在有网络的环境中下载以下关键依赖包（版本需匹配CentOS 8）：

* 基础依赖：`gcc-c++`, `bison`, `flex`, `readline-devel`, `zlib-devel`, `openssl-devel`

* 扩展依赖：`libicu-devel`, `tcl-devel`, `perl-devel`, `python3-devel`, `libxml2-devel`, `libxslt-devel`, `openldap-devel`

* 文档构建：`docbook-dtds`, `docbook-style-xsl`, `xmlto`, `fop`

* 创建依赖包目录（需在联网环境中执行）

```shell
mkdir -p /usr/local/postgresql/depends
```

* 下载所有必要依赖

```shell 
dnf download --resolve --destdir=/usr/local/postgresql/depends \
    gcc-c++ bison flex readline-devel zlib-devel openssl-devel \
    libicu-devel tcl-devel perl-devel python3-devel libxml2-devel \
    libxslt-devel openldap-devel docbook-dtds docbook-style-xsl xmlto fop \
    pam-devel gcc make cmake lz4-devel libuuid-devel systemd-devel
```

* 打包整个目录

```shell
tar -czf postgresql_depends_offline_packages.tar.gz /usr/local/postgresql/depends
```

## 五、配置数据库实例 ##

### 1、安装依赖 ###

* 将下载到 `/usr/local/postgresql/depends` 文件夹下的依赖上传的离线服务器（必做）

* 解压离线包

```shell
tar -xzf postgresql_depends_offline_packages.tar.gz -C /
```

* 使用 DNF 离线安装所有 RPM 依赖包（自动处理依赖顺序）

```shell
dnf install -y /usr/local/postgresql/depends/*.rpm
```

* 验证关键依赖是否安装成功

```shell
rpm -q perl-ExtUtils-Embed readline-devel zlib-devel pam-devel libxml2-devel libxslt-devel openldap-devel python3-devel gcc-c++ openssl-devel cmake
```

### 2、数据库安装 ###

* 切换用户

```shell
su postgresqldba
```

* 进入 `/usr/local/postgresql-18.3` 文件夹

```shell
cd /usr/local/postgresql-18.3
```

* 配置编译选项

```shell
./configure --prefix=/usr/local/postgresql \
            --with-openssl \
			--with-pgport=2345 \
			--with-tcl \
			--with-perl \
			--with-python \
            --with-libxml \
            --with-libxslt \
            --with-icu \
            --with-lz4 \
            --with-zstd \
			--with-uuid=e2fs \
			--with-pam \
			--with-ldap \
			--with-systemd
```

* 编译（-j 参数根据 CPU 核心数调整以加速编译）

```shell
make -j$(nproc)
```

* 安装

```shell
make install
```

### 3、配置环境变量 ### 

```shell
cat >> ~/.bash_profile <<'EOF'
export PGHOME=/usr/local/postgresql
export PGDATA=/usr/local/postgresql/data/
export PATH=$PGHOME/bin:$PATH
export LD_LIBRARY_PATH=$PGHOME/lib:$LD_LIBRARY_PATH
EOF
```

```shell
source ~/.bash_profile
```
### 4、初始化数据库 ###

* 切换到 postgresdba 用户执行初始化

```shell
su - postgresdba
```

* 执行初始化命令，明确指定 -U postgres

```shell
/usr/local/postgresql/bin/initdb -D /usr/local/postgresql/data -U postgres -E UTF-8 --locale=zh_CN.utf8
```

### 5、配置 PostgreSQL 服务 ###

```shell
vim /usr/local/postgresql/data/postgresql.conf
```

##### 按一下键盘字母 `i` 进行编辑 #####

##### 修改以下配置项（根据实际需求调整）： #####

```conf
listen_addresses = '*'          # 允许远程连接（如仅本地连接可保留 localhost）
port = 2345                     # 默认端口，可按需修改
max_connections = 100           # 最大连接数
shared_buffers = 128MB          # 共享缓冲区大小（建议为系统内存的 25%）
password_encryption = scram-sha-256
```

##### 按一下`esc`键 退出编辑 #####

##### `:wq` 保存退出 #####

### 6、配置文件夹权限 ###

* 1. 确保 `/usr/local/postgresql` 目录本身可被访问

```shell
chmod 755 /usr/local/postgresql
```

* 2. 强制递归修改 `data` 目录的所有权（这是最关键的）

```shell
chown -R postgresdba:postgresdb /usr/local/postgresql/data
```

* 3. 确保 `data` 目录权限严格为 `700`

```shell
chmod 700 /usr/local/postgresql/data
```

## 六、创建 Systemd 服务文件，实现 PostgreSQL 开机自启和服务管理 ##

### 1、切换用户 ###

```shell
su - root
```

* 输入 `root` 用户密码

```shell
vim /etc/systemd/system/postgresql.service
```

##### 按一下键盘字母`i`进行编辑 #####

#### 输入以下内容： ####

```shell
[Unit]
Description=PostgreSQL 18.3 Database Server
After=network.target

[Service]
Type=forking
User=postgresdba
Group=postgresdb
ExecStart=/usr/local/postgresql/bin/pg_ctl -D /usr/local/postgresql/data start -w -t 300
ExecStop=/usr/local/postgresql/bin/pg_ctl -D /usr/local/postgresql/data stop -m fast
ExecReload=/usr/local/postgresql/bin/pg_ctl -D /usr/local/postgresql/data reload
TimeoutSec=300

[Install]
WantedBy=multi-user.target
```

##### 按一下`esc`键 退出编辑 #####

##### `:wq` 保存退出 #####

##### 添加文件可执行权限 #####

```Shell
chmod +x /etc/systemd/system/postgresql.service
```

##### 重新加载 `daemon 服务` #####

```shell
sudo systemctl daemon-reload
```

##### 查看 `postgresql.service` 服务是否在服务配置中 #####

```shell
sudo systemctl list-dependencies postgresql.service
```

```shell
sudo systemctl status postgresql.service
```

##### 设置 `postgresql.service` 开机自启 #####

```shell
sudo systemctl enable postgresql.service
```

### 2、快捷启动与关闭 ###

##### 启动 #####

```shell
sudo systemctl start postgresql.service
```

##### 关闭 #####

```shell
sudo systemctl stop postgresql.service
```

##### 重启 #####

```shell
sudo systemctl restart postgresql.service
```

##### 查看状态 #####

```shell
sudo systemctl status postgresql.service
```

### 3、重启计算机 ###

```shell
shutdown -r now
```

### 4、验证是否安全启动 ###

```shell
sudo systemctl status postgresql.service
```

## 七、设置远程访问 ##

### 1、重置密码 ###

##### 切换到 postgresdba 用户 #####

```shell
su - postgresdba
```

##### 指定端口连接 #####

```shell
/usr/local/postgresql/bin/psql -p 2345 -U postgres
```

##### 设置密码 #####

```shell
ALTER USER postgres WITH PASSWORD 'root';
```

##### 退出 #####

```shell
\q
```

### 2、配置远程访问 ###

```shell
vim /usr/local/postgresql/data/pg_hba.conf
```

##### 按一下键盘字母 `i` 进行编辑 #####

##### 在文件末尾添加（允许局域网内所有 IP 使用密码连接）： #####

```conf
# TYPE  DATABASE        USER            ADDRESS                 METHOD
host    all             all             0.0.0.0/0               scram-sha-256
```

##### 按一下`esc`键 退出编辑 #####

##### `:wq` 保存退出 #####

##### 查看防火墙状态 #####

```shell
firewall-cmd --state
```

##### Add 添加开放端口 #####

```shell
firewall-cmd --permanent --zone=public --add-port=2345/tcp
```

##### Reload 重新加载 #####

```shell
firewall-cmd --reload
```

##### 检查是否生效 ######

```shell
firewall-cmd --zone=public --query-port=2345/tcp
```
