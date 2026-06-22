# Centos8 `AMD64` 环境下 `FRP` (内网穿透工具)客户端安装配置 #

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

> 下面的是 `0.61.1` 版本下载地址，需要下载最新的 请手动调整版本号
 
```shell
wget https://github.com/fatedier/frp/releases/download/v0.61.1/frp_0.61.1_linux_arm64.tar.gz
```


### 解压（找到压缩文件） ###

> 下面的是 `v0.61.1` 版本的解压命令，请依据自己下载的版本手动调整版本号

```shell
tar -zxvf frp_0.61.1_linux_amd64.tar.gz -C /usr/local/
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

### 删除服务端文件（可忽略步骤） ###

> 压缩包内的文件包括客户端跟服务端的文件，我们这里是部署客户端，所以只需保留服务端文件（`frpc` 开头文件）其他删掉即可。

#### 切换到目录下 ####

> 下面的是 `v0.61.1` 版本的脚本，请依据自己下载的版本手动调整版本号

```shell
cd /usr/local/frp_0.61.1_linux_amd64/
```

#### 删除服务端文件 ####

```shell
rm -f  rm *frps*
```

### 创建主目录 ###

```shell
mkdir -p /usr/local/frp_linux_amd64
```

### 复制服务文件到主目录 ### 

> 下面的是 `v0.61.1` 版本的脚本，请依据自己下载的版本手动调整版本号

```shell
cp /usr/local/frp_0.61.1_linux_amd64/frpc /usr/local/frp_linux_amd64
```

## 三、配置环境变量 ###

> 从 V0.52.0 版本开始，FRP 开始支持 TOML、YAML 和 JSON 作为配置文件格式。

#### 1、创建配置文件 ####

```shell
touch /usr/local/frp_linux_amd64/frpc.toml
```

#### 2、编辑配置文件 ####

```shell
vim /usr/local/frp_linux_amd64/frpc.toml
```

##### 按一下键盘字母`i`进行编辑 #####

#### 3、输入以下内容： ####

> 注意修改 `server_addr` `server_port` `token` 

```toml
# [common] 整体部分
[common]
# 服务端地址
server_addr = 0.0.0.0
# 服务端注册端口 
server_port = 7000
# 控制台或真实日志文件路径
log_file = ./frpc.log
# 日志级别 trace, debug, info, warn, error
log_level = info
# 日志保留时间
log_max_days = 3
# 指定使用何种身份验证方法将 客户端 与 服务端 进行身份验证。
authentication_method = token
# 否在发送给 服务端 的心跳中包含身份验证令牌
authenticate_heartbeats = true
# 是否在发送到 服务端 的新工作连接中包含身份验证令牌
authenticate_new_work_conns = true
# token 值 由服务端提供
token = 12345678

# 自定义代理名称
[ssh]
# tcp | udp | http | https | stcp | xtcp
type = tcp
# 需要代理的 本地地址
local_ip = 127.0.0.1
# 需要代理的 本地端口
local_port = 22
# 此代理限制带宽，单位为KB和MB
bandwidth_limit = 1MB
# 限制带宽的位置，可选 'client' 客户端或者 'server' 服务端
bandwidth_limit_mode = client
# 客户端与服务端之间的消息是否加密
use_encryption = false
# 消息是否压缩
use_compression = false
# 服务端 远程端口监听
remote_port = 6001
```

##### 按一下`esc`键 退出编辑 #####

##### `:wq` 保存退出 #####

## 四、启动测试 ##

#### 后台静默启动 ####

```shell
nohup ./frpc -c ./frpc.toml &
```

#### 关闭服务 ####

> 如要关闭，使用 `ps aux` 查看进程号，使用 `kill` 即可关闭

## 五、设置`FRP`开机自启 ###

#### 1、 创建 `FRP` 服务文件： ####

```shell
sudo vim /etc/systemd/system/frpc.service
```

##### 按一下键盘字母`i`进行编辑 #####

#### 2、输入以下内容： ####

```shell
[Unit]
Description= FRPC startup script（客户端启动脚本）
After=network.target
After=systemd-user-sessions.service
After=network-online.target

[Service]
ExecStart=/usr/local/frp_linux_amd64/frpc -c /usr/local/frp_linux_amd64/frpc.toml

[Install]
WantedBy=multi-user.target
```

##### 按一下`esc`键 退出编辑 #####

##### `:wq` 保存退出 #####

#### 3、设置其权限为 `775` ####

```shell
sudo chmod 775 /etc/systemd/system/frpc.service
```

#### 4、输入如下命令使`FRPC`服务开机启动  ####

```shell
sudo systemctl enable --now frpc
```

## 六、运维 `FRPC` 

### 1、常用指令

> 启动 `FRPC` 

```shell
sudo systemctl start frpc
```

> 停止 `FRPC` 

```shell
sudo systemctl stop frpc
```

> 重启 `FRPC` 

```shell
sudo systemctl restart frpc
```

> 查看 `FRPC` 状态

```shell
sudo systemctl status frpc
```

### 2、版本更新

#### ①、 停止 `FRPC` 服务： ####

```shell
sudo systemctl stop frpc
```

#### ②、下载最新版本 `FRPC` 服务： ####

> [寻找最新版本安装包](https://github.com/fatedier/frp/releases)

> 载最新安装包 请手动调整版本号
 
```shell
wget https://github.com/fatedier/frp/releases/download/v版本号/frp_版本号_linux_arm64.tar.gz
```

> 下载完成后跟前面安装步骤一致，解压安装包，请依据自己下载的版本手动调整版本号

```shell
tar -zxvf frp_版本号_linux_amd64.tar.gz -C /usr/local/
```

> 复制服务文件到主目录，请依据自己下载的版本手动调整版本号

```shell
cp /usr/local/frp_版本号_linux_amd64/frpc /usr/local/frp_linux_amd64
```

#### ③、启动 `FRPC` 服务： ####

```shell
sudo systemctl start frpc
```

#### ④、查看 `FRPC` 服务状态： ####

```shell
sudo systemctl status frpc
```

#### ④、删除 `FRPC` 安装包： ####

> 删除解压后文件

```shell
rm -rf /usr/local/frp_版本号_linux_amd64
```

> 删除压缩包

```shell
rm -f /usr/local/frp_版本号_linux_amd64.tar.gz
```
