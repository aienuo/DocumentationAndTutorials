# Centos8 环境下 X86 版本 docker-29.1.3 的安装配置 #

### 注意看我的标题！！！！我这是针对 X86 版本的 docker-29.1.3 离线安装

## 一、检查本地是否安装 ##

### 1、检查 ###

```shell

```

### 2、卸载 ###

```shell

```	

## 二、下载 ##

### 1、下载 ###

[docker-29.1.3 下载地址](https://download.docker.com/linux/static/stable/x86_64/docker-29.1.3.tar.gz)

### 2、将下载后的文件上传到 `/usr/local` 文件夹下 ###

* 使用 [windows Terminal 工具](https://apps.microsoft.com/detail/9n8g5rfz9xk3?hl=zh-CN&gl=CN)。 语法： scp 【本地文件地址】 【服务器账号】@【服务器IP地址】:/usr/local/

```shell
scp D:/Downloads/docker-29.1.3.tgz root@100.110.111.108:/usr/local/
```

## 三、解压操作文件 ##

### 1、远程登录服务器 ###

* 使用 [windows Terminal 工具](https://apps.microsoft.com/detail/9n8g5rfz9xk3?hl=zh-CN&gl=CN)。 语法： ssh 【服务器账号】@【服务器IP地址】

```shell
ssh root@100.110.111.108
```

### 2、进入目录解压 `tar.gz` ###

```shell
cd /usr/local/
```

```shell
tar -zxvf docker-29.1.3.tgz  -C /usr/local/
```

## 四、安装 ##

### 1、配置环境变量 ###

#### 修改配置文件 ####

```shell
vim /etc/profile
```

##### 按一下键盘字母`i`进行编辑 #####

#### 输入以下内容： ####

```shell
DOCKER_HOME=/usr/local/docker
PATH=$PATH:$DOCKER_HOME
export DOCKER_HOME
```

##### 按一下`esc`键 退出编辑 #####

##### `:wq` 保存退出 #####

#### 修改立即生效： ####

```shell
source /etc/profile
```

### 2、创建配置文件 ###

#### 创建文件夹 ####

```shell
mkdir -p /etc/docker
```

#### 创建文件 ####

```shell
touch /etc/docker/daemon.json
```

#### 编辑文件 ####

```shell
vim /etc/docker/daemon.json
```

##### 按一下键盘字母`i`进行编辑 #####

#### 输入以下内容： ####

```
{
    "userland-proxy-path": "/usr/local/docker/docker-proxy",
	"storage-driver": "overlay2",
    "data-root": "/usr/local/docker/docker-data",
    "registry-mirrors": [
        "https://docker.1ms.run",
        "https://docker.xuanyuan.me",
		"https://docker.mirrors.tuna.tsinghua.edu.cn"
    ],
	"hosts": [
		"unix:///var/run/docker.sock",
		"tcp://0.0.0.0:6732"
	],
	"tls": true,
	"tlsverify": true,
	"tlscacert": "/usr/local/docker/cert/ca.pem",
	"tlscert": "/usr/local/docker/cert/server-cert.pem",
	"tlskey": "/usr/local/docker/cert/server-key.pem",
    "log-driver": "json-file",
    "log-opts": {
        "max-size": "10m",
        "max-file": "3"
    },
    "live-restore": true
}
```
* docker-proxy 用于实现 Docker 容器端口映射时的用户态代理
* storage-driver Docker 数据存储存动类型
* data-root Docker 数据存储目录
* registry-mirrors Docker 镜像仓库
* hosts 定义 Docker 守护进程监听的端点
* hosts.unix 本地 Unix 套接字，供同一主机上的客户端（如 docker 命令）使用
* hosts.tcp 监听所有网络接口的 TCP 6732 端口，允许远程客户端连接
* tls 启用 TLS（传输层安全），表示使用加密通信。
* tlsverify 启用客户端验证，强制客户端必须提供有效证书
* tlscacert 指定 CA 证书路径，用于验证客户端证书的签发者。该 CA 必须与客户端证书的签发 CA 一致。
* tlscert 服务器端证书，用于向客户端证明自己的身份。
* tlskey 服务器端私钥，用于 TLS 握手时解密数据
* log-driver 日志存储为 JSON 文件
* log-opts.max-size 单个日志文件最大 10MB
* log-opts.max-file 最多保留 3 个日志文件
* live-restore 更新daemon.json配置文件时，自动加载配置，不用重新启动Docker

##### 按一下`esc`键 退出编辑 #####

##### `:wq` 保存退出 #####

### 3、测试是否安装成功 ###

```shell
docker -v
```

```shell
docker version
```

* 创建日志文件夹

```shell
mkdir -p /usr/local/docker/log
```


### 4、配置远程连接加密证书 ###

#### 创建文件 ####

```shell
touch /usr/local/docker/create_certificate.sh
```

#### 编辑文件 ####

```shell
vim /usr/local/docker/create_certificate.sh
```

##### 按一下键盘字母`i`进行编辑 #####

#### 输入以下内容： ####

```shell
#!/bin/sh
# 服务器IP
ip=
# 证书密码
password=
# 证书生成位置
dir=/usr/local/docker/cert
# 证书有效期10年
validity_period=3660

# 将此 shell 脚本在安装 docker 的机器上执行，作用是生成 docker 远程连接加密证书
if [ ! -d "$dir" ]; then
  echo ""
  echo "$dir , not dir , will create"
  echo ""
  mkdir -p $dir
else
  echo ""
  echo "$dir , dir exist , will delete and create"
  echo ""
  rm -rf $dir
  mkdir -p $dir
fi

cd $dir || exit
# 创建根证书RSA私钥
openssl genrsa -aes256 -passout pass:"$password" -out ca-key.pem 4096
# 创建CA证书
openssl req -new -x509 -days $validity_period -key ca-key.pem -passin pass:"$password" -sha256 -out ca.pem -subj "/C=NL/ST=./L=./O=./CN=$ip"
# 创建服务端私钥
openssl genrsa -out server-key.pem 4096
# 创建服务端签名请求证书文件
openssl req -subj "/CN=$ip" -sha256 -new -key server-key.pem -out server.csr

echo subjectAltName = IP:$ip,IP:0.0.0.0 >>extfile.cnf

echo extendedKeyUsage = serverAuth >>extfile.cnf
# 创建签名生效的服务端证书文件
openssl x509 -req -days $validity_period -sha256 -in server.csr -CA ca.pem -CAkey ca-key.pem -passin "pass:$password" -CAcreateserial -out server-cert.pem -extfile extfile.cnf
# 创建客户端私钥
openssl genrsa -out key.pem 4096
# 创建客户端签名请求证书文件
openssl req -subj '/CN=client' -new -key key.pem -out client.csr

echo extendedKeyUsage = clientAuth >>extfile.cnf

echo extendedKeyUsage = clientAuth >extfile-client.cnf
# 创建签名生效的客户端证书文件
openssl x509 -req -days $validity_period -sha256 -in client.csr -CA ca.pem -CAkey ca-key.pem -passin "pass:$password" -CAcreateserial -out cert.pem -extfile extfile-client.cnf
# 删除多余文件
rm -f -v client.csr server.csr extfile.cnf extfile-client.cnf

# 防止密钥文件被误删或者损坏，改变文件权限，让它只读
chmod -v 0400 ca-key.pem key.pem server-key.pem

# 防止证书损坏，改变文件权限，让它只读
chmod -v 0444 ca.pem server-cert.pem cert.pem
```

##### 按一下`esc`键 退出编辑 #####

##### `:wq` 保存退出 #####

#### 赋权 ####

```shell
chmod 777 /usr/local/docker/create_certificate.sh
```

#### 切换文件夹 ####

```shell
cd /usr/local/docker/
```

#### 执行并生成密钥 ####

```shell
./create_certificate.sh
```

### 5、编写 `docker.service` 文件加入 `Linux 服务` 当中并开启守护进程 ###

```shell
vim /etc/systemd/system/docker.service
```

##### 按一下键盘字母 `i` 进行编辑 #####

##### 输入以下内容： #####

```
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network-online.target firewalld.service
Wants=network-online.target
 
[Service]
Environment="PATH=/usr/local/docker:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
Type=notify
ExecStart=/usr/local/docker/dockerd
ExecReload=/bin/kill -s HUP $MAINPID
LimitNOFILE=infinity
LimitNPROC=infinity
TimeoutStartSec=0
Delegate=yes
KillMode=process
Restart=on-failure
StartLimitBurst=3
StartLimitInterval=60s

[Install]
WantedBy=multi-user.target
```

##### 按一下`esc`键 退出编辑 #####

##### `:wq` 保存退出 #####

##### 添加文件可执行权限 #####

```Shell
chmod +x /etc/systemd/system/docker.service
```

##### 重新加载 `daemon 服务` #####

```shell
sudo systemctl daemon-reload
```

##### 查看 `docker.service` 服务是否在服务配置中 #####

```shell
sudo systemctl status docker.service
```

##### 设置 `docker.service` 开机自启 #####

```shell
sudo systemctl enable docker.service
```

### 6、快捷启动与关闭 ###

##### 启动 #####

```shell
sudo systemctl start docker.service
```

##### 关闭 #####

```shell
sudo systemctl stop docker.service
```

##### 重启 #####

```shell
sudo systemctl restart docker.service
```

##### 查看状态 #####

```shell
sudo systemctl status docker.service
```

### 6、重启计算机 ###

```shell
shutdown -r now
```

##### 远程登录服务器 #####

* 使用 [windows Terminal 工具](https://apps.microsoft.com/detail/9n8g5rfz9xk3?hl=zh-CN&gl=CN)。 语法： ssh 【服务器账号】@【服务器IP地址】

```shell
ssh root@100.110.111.102
```

##### 验证是否安全启动 #####

```shell
sudo systemctl status docker.service
```

```shell
ss -tuln | grep 6732
```

```shell
docker -H 100.110.111.108:6732 ps
```

### 7、防火墙端口开放 ###

##### 查看防火墙状态 #####

```shell
firewall-cmd --state
```

##### 开启防火墙 #####

```shell
systemctl start firewalld
```
	
##### Add 添加开放端口 #####

```shell
firewall-cmd --permanent --zone=public --add-port=6732/tcp
```

##### Reload 重新加载 #####

```shell
firewall-cmd --reload
```

##### 检查是否生效 ######

```shell
firewall-cmd --zone=public --query-port=6732/tcp
```

### 8、使用 IDEA 的 Docker 插件 远程链接 ###

#### 开黑窗口下载生成的密钥文件 ####

* 使用 [windows Terminal 工具](https://apps.microsoft.com/detail/9n8g5rfz9xk3?hl=zh-CN&gl=CN)。 语法： scp 【服务器账号】@【服务器IP地址】:"【文件1】 【文件2】 【...】" 【本地目录】

```shell
scp root@100.110.111.108:/usr/local/docker/cert/ca.pem D:\Downloads\
```

```shell
scp root@100.110.111.108:/usr/local/docker/cert/cert.pem D:\Downloads\
```

```shell
scp root@100.110.111.108:/usr/local/docker/cert/key.pem D:\Downloads\
```

#### 下载合适的 Docker 执行文件 ###

* [点击下载 Docker](https://download.docker.com/win/static/stable/x86_64/)

#### 















## （可选配置）五、下载安装配置 `Docker Compose` ##

### 1、下载 ###

[Docker Compose 下载地址](https://github.com/docker/compose/releases)

### 2、将下载后的文件上传到 `/usr/local` 文件夹下 ###

* 使用 [windows Terminal 工具](https://apps.microsoft.com/detail/9n8g5rfz9xk3?hl=zh-CN&gl=CN)。 语法： scp 【本地文件地址】 【服务器账号】@【服务器IP地址】:/usr/local/

```shell
scp D:/Downloads/docker-compose-darwin-x86_64 root@100.110.111.102:/usr/local/docker/
```

### 3、远程登录服务器 ###

* 使用 [windows Terminal 工具](https://apps.microsoft.com/detail/9n8g5rfz9xk3?hl=zh-CN&gl=CN)。 语法： ssh 【服务器账号】@【服务器IP地址】

```shell
ssh root@100.110.111.102
```

### 4、修改文件名 ###

```shell
cd /usr/local/docker
```

```shell
mv docker-compose-darwin-x86_64 docker-compose
```

### 5、赋予执行权限 ###

```shell
chmod +x /usr/local/docker/docker-compose
```

### 6、创建软链接（可选） ###

```shell
ln -s /usr/local/docker/docker-compose /usr/bin/docker-compose 
```

### 7、验证版本 ###

```shell
docker-compose --version  # 应输出 Docker Compose version
```
