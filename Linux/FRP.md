# Centos8 `AMD64` 环境下 `FRP` (内网穿透工具)服务端安装配置 #

### 注意看我的标题！！！！我这是针对 `AMD64`  ###

> [寻找最新版本安装包](https://github.com/fatedier/frp/releases)

## 一、检查本地是否安装 ##

### 1、检查 ###

```shell
rpm -qa | grep frp
```

### 2、卸载 ###

```shell
rpm -e 已经存在的FRP全名
```

## 二、解压操作文件 ##

### 下载并上传到 /usr/local/ 文件夹下

#### 查看 CPU 架构 ####

```shell
arch
```

|        输出        | 对应架构  |
|:----------------:|:-----:|
|       mips       | MIPS  |
| x86_64，x64，AMD64 | amd64 |
|  aarch64， arm64  | ARM64 |

#### 下载地址 ####

> 下面的是 `v0.51.2` 版本下载地址，需要下载最新的 请手动调整版本号
 
```shell
wget https://github.com/fatedier/frp/releases/download/v0.51.2/frp_0.51.2_linux_arm64.tar.gz
```

### 解压（找到压缩文件） ###

> 下面的是 `v0.51.2` 版本下载地址，需要下载最新的 请手动调整版本号

```shell
tar -zxvf frp_0.51.2_linux_amd64.tar.gz -C /usr/local/
```

#### 切换到解压目录下 ####

```shell
cd /usr/local/
```

#### 查看是否剪切成功 ####

```shell
ls -a
ll
```

### 删除客户端文件（可忽略步骤） ###

> 压缩包内的文件包括客户端跟服务端的文件，我们这里是部署服务器端，所以只需保留服务端文件（`frps` 开头文件）其他删掉即可。

#### 切换到目录下 ####

> 下面的是 `v0.51.2` 版本下载地址，需要下载最新的 请手动调整版本号

```shell
cd /usr/local/frp_0.51.2_linux_amd64/
```

#### 删除客户端文件 ####

```shell
rm -f  rm *frpc*
```

### 创建主目录 ###

```shell
mkdir -p /usr/local/frp_linux_amd64
```

### 复制服务文件到主目录 ### 

> 下面的是 `v0.51.2` 版本下载地址，需要下载最新的 请手动调整版本号

```shell
cp /usr/local/frp_0.51.2_linux_amd64/frps /usr/local/frp_linux_amd64
```

## 三、配置环境变量 ###

#### 1、创建配置文件 ####

```shell
touch /usr/local/frp_linux_amd64/frps.ini
```

#### 2、编辑配置文件 ####

> 参考 [frps_full.ini](frps_full.ini) 文件

```shell
vim /usr/local/frp_linux_amd64/frps.ini
```

##### 按一下键盘字母`i`进行编辑 #####

#### 3、输入以下内容： ####

```shell
[common]
bind_addr = 0.0.0.0
bind_port = 7000
kcp_bind_port = 7000

vhost_http_port = 80
vhost_https_port = 443

dashboard_addr = 0.0.0.0
dashboard_port = 7500
dashboard_user = admin
dashboard_pwd = admin@12345
dashboard_tls_mode = false
enable_prometheus = true

log_file = /usr/local/frp_linux_amd64/frps.log
# trace, debug, info, warn, error （跟踪、调试、信息、警告、错误）
log_level = error
log_max_days = 3

authentication_timeout = 0
```

##### 按一下`esc`键 退出编辑 #####

##### `:wq` 保存退出 #####

### 四、启动测试 ###

#### 后台静默启动 ####

```shell
nohup ./frps -c ./frps.ini &
```

#### 关闭服务 ####

> 如要关闭，使用 `ps aux` 查看进程号，使用 `kill` 即可关闭

#### 查看服务端可视化面板 ####

> 服务开启后，在任意浏览器访问 URL: `http://服务端公网IP:7500`，输入用户名和密码，即可打开可视化面板

### 五、设置`FRP`开机自启 ###

#### 1、 创建 `FRP` 服务文件： ####

```shell
sudo vim /etc/systemd/system/frp.service
```

##### 按一下键盘字母`i`进行编辑 #####

#### 2、输入以下内容： ####

```shell
[Unit]
Description= FRP startup script（启动脚本）
After=network.target
After=systemd-user-sessions.service
After=network-online.target

[Service]
ExecStart=/usr/local/frp_linux_amd64/frps -c /usr/local/frp_linux_amd64/frps.ini

[Install]
WantedBy=multi-user.target
```

##### 按一下`esc`键 退出编辑 #####

##### `:wq` 保存退出 #####

#### 3、设置其权限为 `775` ####

```shell
sudo chmod 775 /etc/systemd/system/frp.service
```

#### 4、输入如下命令使`FRP`服务开机启动  ####

```shell
sudo systemctl enable --now frp
```
