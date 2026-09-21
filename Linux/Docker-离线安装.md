# Centos8 环境下 X86 版本 docker-29.1.3 的安装配置 #

### 注意看我的标题！！！！我这是针对 X86 版本的 docker-29.1.3 离线安装

## 一、检查本地是否安装 ##

### 1、检查 ###

```shell
docker version
```

### 2、卸载 ###

> 删除 docker 及其相关文件

## 二、下载 ##

### 1、下载 ###

[docker-29.1.3 下载地址](https://download.docker.com/linux/static/stable/x86_64/docker-29.1.3.tar.gz)

### 2、将下载后的文件上传到 `/usr/local` 文件夹下 ###

* 使用 [windows Terminal 工具](https://apps.microsoft.com/detail/9n8g5rfz9xk3?hl=zh-CN&gl=CN)。 语法： scp 【本地文件地址】【服务器账号】@【服务器IP地址】:/usr/local/

```shell
scp D:/Downloads/docker-29.1.3.tgz root@100.110.111.108:/usr/local/
```

## 三、解压操作文件 ##

### 1、远程登录服务器 ###

* 使用 [windows Terminal 工具](https://apps.microsoft.com/detail/9n8g5rfz9xk3?hl=zh-CN&gl=CN)。 语法： ssh【服务器账号】@【服务器IP地址】

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

| 存储驱动      | 特点                                                      | 适用场景                                   |
|---------------|-----------------------------------------------------------|--------------------------------------------|
| overlay2      | Docker 默认驱动，基于 Linux OverlayFS，性能好、内存开销低 | 主流现代 Linux 发行版（CentOS、Ubuntu 等） |
| AUFS          | 最早的 Docker 驱动，多层联合文件系统                      | 旧版 Ubuntu/Debian 系统                    |
| Device Mapper | 基于块设备映射器，工作在块级别而非文件级别                | RedHat/CentOS 旧版本                       |
| Btrfs/ZFS     | 高级文件系统，原生支持快照和克隆                          | 特定场景                                   |

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

* 使用 [windows Terminal 工具](https://apps.microsoft.com/detail/9n8g5rfz9xk3?hl=zh-CN&gl=CN)。 语法： ssh【服务器账号】@【服务器IP地址】

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

* 使用 [windows Terminal 工具](https://apps.microsoft.com/detail/9n8g5rfz9xk3?hl=zh-CN&gl=CN)。 语法： scp
  【服务器账号】@【服务器IP地址】:"【文件1】 【文件2】 【...】" 【本地目录】

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

## （可选配置）五、下载安装配置 `Docker Compose` 多容器编排工具 ##

### 1、下载 ###

[Docker Compose 下载地址](https://github.com/docker/compose/releases)

### 2、将下载后的文件上传到 `/usr/local` 文件夹下 ###

* 使用 [windows Terminal 工具](https://apps.microsoft.com/detail/9n8g5rfz9xk3?hl=zh-CN&gl=CN)。 语法： scp 【本地文件地址】【服务器账号】@【服务器IP地址】:/usr/local/

```shell
scp D:/Downloads/docker-compose-darwin-x86_64 root@100.110.111.102:/usr/local/docker/
```

### 3、远程登录服务器 ###

* 使用 [windows Terminal 工具](https://apps.microsoft.com/detail/9n8g5rfz9xk3?hl=zh-CN&gl=CN)。 语法： ssh【服务器账号】@【服务器IP地址】

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

### 7、`Compose` 文件结构 ###

> 推荐文件名：`compose.yaml` 或 `docker-compose.yml` （配置文件内冒号隔开的左主右容，前面是主机，后面是容器）

```yaml
services:
  web-server:
    build: .
    ports:
      - "80:80" # 宿主机80 → 容器80
    depends_on:
      api-server:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "wget", "-q", "--spider", "http://localhost/"]
      interval: 30s
      timeout: 5s
      retries: 3
      start_period: 15s

  api-server:
    build: ./api
    ports:
      - "8080:8080" # 宿主机8080 → 容器8080
    environment:
      DB_HOST: postgres-server
      DB_PORT: 2345
      DB_USER: user_api_server
      DB_PASSWORD: secret
      DB_NAME: server_config
    healthcheck:
      test: ["CMD", "wget", "-q", "--spider", "http://localhost:8080/health"]
      interval: 30s
      timeout: 5s
      retries: 3
      start_period: 30s
    depends_on:
      postgres-server:
        condition: service_healthy

  postgres-server:
    image: postgres:14
    ports:
      - "2345:5432" # 宿主机2345 → 容器 5432
    environment:
      POSTGRES_PASSWORD: secret
      POSTGRES_USER: user_api_server
      POSTGRES_DB: server_config
    volumes:
      - db-data:/var/lib/postgresql/data  # 宿主机路径 → 容器路径
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user_api_server -d server_config"]
      interval: 5s
      timeout: 3s
      retries: 10
      start_period: 10s

volumes:
  db-data:
    driver: local
```

### 8、`Docker Compose` 常用命令 ###

| 命令                                  | 作用                 |
|---------------------------------------|----------------------|
| `docker compose up -d`                | 后台启动所有服务     |
| `docker compose down`                 | 停止并删除容器、网络 |
| `docker compose logs -f`              | 查看聚合日志         |
| `docker compose ps`                   | 查看服务状态         |
| `docker compose build`                | 构建所有服务镜像     |
| `docker compose exec web-server bash` | 进入运行中的服务     |
| `docker compose config`               | 验证合并后的配置     |

## 六、`Docker` 常用操作指令 ##

### 1、版本查看 ###

```shell
docker -v
```

```shell
docker version
```

### 2、查看 `docker` 内的镜像  ###

* 查看列表

```shell
docker images ls
```

* 名称检索

```shell
docker images | grep 【镜像名】
```

示例：

```shell
docker images | grep nginx_image
```

```shell
docker images | grep -E "nginx|redis|mysql"
```

### 3、导出 `docker` 镜像到本地 `tar` 文件  ###

* 原生写法

```shell
docker save --output /usr/local/docker/tar_files/【镜像名】.tar 【要导出的镜像名】:【标签】
```

* 用 `-o` 代替 `--output`

```shell
docker save -o /usr/local/docker/tar_files/【镜像名】.tar 【要导出的镜像名】:【标签】
```

* 用重定向代替 `-o`

```shell
docker save 【要导出的镜像名】:【标签】 > /usr/local/docker/tar_files/【镜像名】.tar
```

示例：

```shell
docker save -o /usr/local/docker/tar_files/nginx_image.tar nginx:1.25
```

* ### 导出多个 `docker` 镜像到本地 `tar` 文件 ###

#### 导出指定 `docker` 镜像 ####

```shell
docker save -o /usr/local/docker/tar_files/【自定义合集名】.tar 【要导出的镜像名】:【标签】 【要导出的镜像名】:【标签】 【要导出的镜像名】:【标签】
```

示例：
```shell
docker save -o /usr/local/docker/tar_files/multiple_images.tar nginx:1.25 mysql:8.4.5 redis:6.2.6
```

#### 导出所有 `docker` 镜像 ####

```shell
docker save -o /usr/local/docker/tar_files/【自定义合集名】.tar $(docker images -q)
```

示例：

```shell
docker save -o /usr/local/docker/tar_files/multiple_images.tar $(docker images -q)
```

### 4、查看 `tar` 文件内部结构 ###

```shell
tar -tf /usr/local/docker/tar_files/【镜像名】.tar | head
```

示例：

```shell
tar -tf /usr/local/docker/tar_files/nginx_image.tar | head
```

### 5、查看 `tar` 文件大小 ###

```shell
ls -lh /usr/local/docker/tar_files/【镜像名】.tar
```

示例：

```shell
tar -tf /usr/local/docker/tar_files/nginx_image.tar | head
```

### 6、查看 `tar` 里包含哪些镜像 ###

```shell
tar -tf 【镜像名】.tar | grep manifest.json
```

示例：

```shell
tar -tf multiple_images.tar | grep manifest.json
```

### 7、查看 `tar` 文件架构与 OS信息 ###

```shell
CFG=$(tar -xOf /usr/local/docker/tar_files/【镜像名】.tar manifest.json | python3 -c "import sys,json;print(json.load(sys.stdin)[0]['Config'])")
tar -xOf /usr/local/docker/tar_files/【镜像名】.tar "$CFG" | python3 -c "import sys,json;d=json.load(sys.stdin);print(d['architecture'], d['os'])"
```

示例：

```shell
CFG=$(tar -xOf /usr/local/docker/tar_files/nginx_image.tar manifest.json | python3 -c "import sys,json;print(json.load(sys.stdin)[0]['Config'])")
tar -xOf /usr/local/docker/tar_files/nginx_image.tar "$CFG" | python3 -c "import sys,json;d=json.load(sys.stdin);print(d['architecture'], d['os'])"
```

### 8、加载本地 `tar` 文件到 `docker` 镜像 ###

* 原生写法

```shell
docker load --input /usr/local/docker/tar_files/【镜像名】.tar
```

* 用 `-i` 代替 `--input`

```shell
docker load -i /usr/local/docker/tar_files/【镜像名】.tar
```

* 用重定向代替 `-i`

```shell
docker load < /usr/local/docker/tar_files/【镜像名】.tar
```

示例：

```shell
docker load < /usr/local/docker/tar_files/nginx_image.tar
```

### 9、根据 `docker` 镜像运行 `docker` 容器 ###

```shell
docker run -d \
  --name 【容器名称】 \
  --log-driver json-file \
  --log-opt max-size=10m \
  --log-opt max-file=3 \
  --restart always \
  -p 【端口映射】 \
  -e 【环境变量】 \
  -v 【挂载数据卷/目录】 \
  【镜像名】:【标签】
```

| 命令                     | 作用                                                  |
|--------------------------|-------------------------------------------------------|
| `docker run`             | 创建 + 启动容器                                       |
| `-d`                     | 后台运行（detached）                                  |
| `--name`                 | 给容器起名字（保证全局唯一且只能用 [a-zA-Z0-9_.-]）   |
| `--log-driver json-file` | 使用 json 文件作为日志驱动                            |
| `--log-opt max-size=10m` | 单个日志文件最大 10MB，达到上限后自动轮转             |
| `--log-opt max-file=3`   | 最多保留 3 个日志文件（含当前文件），超出后删除最旧的 |
| `--restart`              | 容器退出/重启后自动拉起                               |
| `-p`                     | 端口映射（宿主机:容器）                               |
| `-e`                     | 注入环境变量（key=value）                             |
| `-v`                     | 挂载数据卷/目录（宿主机路径:容器路径[:权限]）         |

示例：

* #### 启动一个 `Nginx` 容器 ####

```shell
docker run -d \  # 后台运行（detached）
  --name nginx_server \  # 容器名称
  --log-driver json-file \  # 日志驱动
  --log-opt max-size=10m \  # 单个日志文件最大 10MB，达到上限后自动轮转
  --log-opt max-file=3 \  # 最多保留 3 个日志文件（含当前文件），超出后删除最旧的
  --restart always \  # 容器退出/重启后自动拉起
  -p 80:80 \  # 端口映射（宿主机:容器）
  -p 443:443 \  # 端口映射（宿主机:容器）
  -e TZ=Asia/Shanghai \  # 设置时区
  -v /usr/local/nginx/html:/usr/share/nginx/html \  # 网页目录挂载
  -v /usr/local/nginx/conf/server:/etc/nginx/conf.d \  # 配置文件目录挂载
  -v /usr/local/nginx/log:/var/log/nginx \  # 日志目录挂载
  -v /usr/local/nginx/conf/nginx.conf:/etc/nginx/nginx.conf:ro \  # 主配置文件挂载，容器内只读
  nginx:1.25  # 镜像名:标签
```

查看容器 `端口映射`

```shell
docker port nginx_server
```

查看容器 `环境变量`

```shell
docker exec nginx_server env
```

```shell
docker inspect nginx_server | grep -A20 Env
```

查看容器 `数据卷挂载`

```shell
docker inspect nginx_server | grep -A10 Mounts
```

* #### 启动一个 `Redis` 容器 ####

```shell
docker run -d \  # 后台运行（detached）
  --name redis_server \  # 容器名称
  --log-driver json-file \  # 日志驱动
  --log-opt max-size=10m \  # 单个日志文件最大 10MB，达到上限后自动轮转
  --log-opt max-file=3 \  # 最多保留 3 个日志文件（含当前文件），超出后删除最旧的
  --restart always \  # 容器退出/重启后自动拉起
  -p 9736:6379 \  # 端口映射（宿主机:容器）
  -e REDIS_PASSWORD=123456 \  # 设置 Redis 密码
  -v /usr/local/data/redis:/data \  # 目录挂载
  redis:6.2.6  # 镜像名:标签
```

* #### 启动一个 `MySQL` 容器 ####

```shell
  docker run -d \  # 后台运行（detached）
  --name mysql_8_server \  # 容器名称
  --log-driver json-file \  # 日志驱动
  --log-opt max-size=10m \  # 单个日志文件最大 10MB，达到上限后自动轮转
  --log-opt max-file=3 \  # 最多保留 3 个日志文件（含当前文件），超出后删除最旧的
  -p 6033:3306 \  # 端口映射（宿主机:容器）
  -e MYSQL_ROOT_PASSWORD=123456 \  # 设置 MySQL root用户 密码
  -e MYSQL_DATABASE=cloud_config \  # 默认创建数据库
  -v /usr/local/mysql/data/mysql_8:/var/lib/mysql \  # 数据目录挂载
  -v /usr/local/mysql/init_db_sql/mysql_8.sql:/docker-entrypoint-initdb.d/init-db.sql \  # 只在数据目录为空（即首次初始化）时执行。如果你已经用旧的容器跑过，/var/lib/mysql 里有数据了，脚本就不会再执行，需要先清空数据重新启动才行
  mysql:8.4.5  # 镜像名:标签
```


### 10、查看 `docker` 容器状态 ###

```shell
docker ps -a
```

* 输出字段：

| 字段         | 说明                                                       |
|--------------|------------------------------------------------------------|
| CONTAINER ID | 容器唯一标识（12位短ID，加 `--no-trunc` 可显示完整64位ID） |
| IMAGE        | 容器基于的镜像名称和标签                                   |
| COMMAND      | 容器启动时执行的命令（来自镜像的 ENTRYPOINT/CMD）          |
| CREATED      | 容器创建至今的时间间隔                                     |
| STATUS       | 容器当前状态（见下方状态详解）                             |
| PORTS        | 端口映射关系（已停止的容器无端口映射）                     |
| NAMES        | 容器名称（未指定时 Docker 自动分配随机名称）               |

* STATUS 状态详解：

| 状态       | 含义                                                              |
|------------|-------------------------------------------------------------------|
| created    | 已创建但未启动（通过 `docker create` 创建）                       |
| running    | 正在运行中                                                        |
| restarting | 正在重启（手动 `docker restart` 或配置了重启策略触发）            |
| removing   | 正在被删除（`docker rm` 执行中）                                  |
| paused     | 已暂停（通过 `docker pause` 暂停，进程冻结但不释放资源）          |
| exited     | 已停止（进程退出，不消耗 CPU/内存，但容器元数据和文件系统仍保留） |
| dead       | 死亡状态（不可操作，只能被删除，通常是删除过程中出现异常导致）    |

* 示例：

```shell
# 只显示已停止的容器
docker ps -a -f "status=exited"

# 只显示正在运行的容器
docker ps -a -f "status=running"

# 只显示重启中的容器
docker ps -a -f "status=restarting"

# 按名称过滤（支持模糊匹配）
docker ps -a -f "name=nginx_server"

# 按镜像过滤
docker ps -a -f "ancestor=nginx:1.25"

# 最近创建的 3 个
docker ps -a -n 3

# 最近创建的 1 个
docker ps -l

# 只显示名称和状态
docker ps -a --format "table {{.Names}}\t{{.Status}}"

# 只显示名称和启动命令
docker ps -a --format "{{.Names}}: {{.Command}}" --no-trunc

# 显示名称、镜像、运行时长、状态
docker ps -a --format "table {{.Names}}\t{{.Image}}\t{{.RunningFor}}\t{{.Status}}"
```

| 占位符            | 含义     |
|-------------------|----------|
| `{{.ID}}`         | 容器 ID  |
| `{{.Image}}`      | 镜像名称 |
| `{{.Command}}`    | 启动命令 |
| `{{.CreatedAt}}`  | 创建时间 |
| `{{.RunningFor}}` | 运行时长 |
| `{{.Status}}`     | 状态     |
| `{{.Ports}}`      | 端口映射 |
| `{{.Names}}`      | 容器名称 |
| `{{.Labels}}`     | 标签     |
| `{{.Mounts}}`     | 挂载卷   |
| `{{.Size}}`       | 容器大小 |

### 11、停止 `docker` 指定容器 ###

* 停止 `docker` 指定容器

```shell
docker stop 【容器名称】
```

示例：

```shell
docker stop nginx_server
```

### 12、启动 `docker` 指定容器 ###

```shell
docker start 【容器名称】
```

示例：

* 启动指定容器

```shell
docker start nginx_server
```

* 批量启动所有已停止的容器

```shell
docker start $(docker ps -a -q -f "status=exited")
```

### 13、重启 `docker` 指定容器（stop + start） ###

```shell
docker restart 【容器名称】
```

示例：

```shell
docker restart nginx_server
```

### 14、查看 `docker` 容器日志 ###

> 查看容器的标准输出（stdout）和标准错误（stderr）日志
> 日志文件默认在宿主机 /var/lib/docker/containers/<容器ID>/<容器ID>-json.log

```shell
docker logs 【容器名称】
```

* 常用参数

| 参数               | 作用                           |
|--------------------|--------------------------------|
| `-f, --follow`     | 实时跟踪输出（类似 `tail -f`） |
| `--tail <n>`       | 只看最后 n 行                  |
| `-t, --timestamps` | 显示时间戳                     |
| `--since <时间>`   | 从某时间之后的日志             |
| `--until <时间>`   | 到某时间为止的日志             |

示例：

```shell
# 实时跟踪（最常用）
docker logs -f nginx_server

# 看最后 100 行
docker logs --tail 100 nginx_server

# 实时跟踪最后 50 行，带时间戳
docker logs -f --tail 50 -t nginx_server

# 看最近 10 分钟的日志
docker logs --since 10m nginx_server

# 看指定时间段的日志
docker logs --since "2026-09-20T10:00:00" --until "2026-09-20T11:00:00" nginx_server
```

### 15、进入正在运行的 `docker` 容器 ###

```shell
docker exec -it 【容器名称】 /bin/bash
```

| 部分          | 含义                          |
|---------------|-------------------------------|
| `docker exec` | 在运行中的容器里执行命令      |
| `-i`          | interactive，保持标准输入打开 |
| `-t`          | tty，分配伪终端               |
| `/bin/bash`   | 要执行的命令（进入 bash）     |

> `-i` 和 `-t` 要合为 `-it`，才能得到一个 **可交互的 shell**
> 
> `-i`：让输入能传进容器；
>
> `-t`：分配一个终端，显示才正常（有提示符、支持彩色）
>
> 只写一个会有问题：

```shell
docker exec -i nginx_server bash    # 能执行但无提示符，体验差
docker exec -t nginx_server bash    # 有终端但输入不进去
```

* 1、很多精简镜像（alpine、distroless）**没有 bash**，会报：

```
exec: "bash": executable file not found in $PATH
```
改用 `sh`：

```shell
docker exec -it nginx_server sh
```
判断有没有 `bash`：

```shell
docker exec nginx_server ls /bin/bash
```

* 2、扩展用法

```shell
# 查看容器内进程
docker exec nginx_server ps aux

# 查看环境变量
docker exec nginx_server env

# 查看目录
docker exec nginx_server ls -l /etc/nginx

# 在容器内执行 Redis 命令
docker exec -it redis_server redis-cli

# 进入 MySQL
docker exec -it mysql_server mysql -uroot -p

# 以 root 身份进入（容器默认用户非 root 时）
docker exec -it -u root nginx_server bash

# 指定工作目录
docker exec -it -w /etc/nginx nginx_server bash
```

| 参数           | 作用                |
|----------------|---------------------|
| `-i`           | 保持 stdin 打开     |
| `-t`           | 分配伪终端          |
| `-u <用户>`    | 指定用户（如 root） |
| `-w <目录>`    | 指定工作目录        |
| `-e KEY=VALUE` | 临时注入环境变量    |
| `-d`           | 后台执行            |

* 3、退出容器

> 退出 shell，容器继续运行（exec 不影响容器）

```shell
exit
```

```shell
Ctrl+D
```

> **重要**：`exec` 进入后 `exit`，**容器不会停止**，因为 exec 起的进程不是容器主进程。这也是它比 `attach` 安全的原因。