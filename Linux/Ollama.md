# `Centos8` 环境下 `Ollama` 的安装配置 #

### 注意看我的标题！！！！[参考资料](https://zhuanlan.zhihu.com/p/704951717/)  ###

## 一、自动安装 ##

```shell
curl -fsSL https://ollama.com/install.sh | sh
```

## 二、手动安装 ##

### 1、下载安装包 ###

```shell
sudo curl -L https://ollama.com/download/ollama-linux-amd64 -o /usr/bin/ollama
```

#### 赋权 ####

```shell
sudo chmod +x /usr/bin/ollama
```

#### 为 `Ollama` 创建一个用户 ####

```shell
sudo useradd -r -s /bin/false -m -d /usr/share/ollama ollama
```

### 2、将 `Ollama` 添加为启动服务 ###

#### 在 `/etc/systemd/system/` 目录下创建一个服务文件 `ollama.service` ####

```shell
vim /etc/systemd/system/ollama.service
```

#### 按一下键盘字母 `I` 进行编辑，追加如下内容 ####

```
[Unit]
Description=Ollama Service
After=network-online.target

[Service]
ExecStart=/usr/bin/ollama serve
User=ollama
Group=ollama
Restart=always
RestartSec=3

[Install]
WantedBy=default.target
```

#### 按一下 `ESC` 键 退出编辑 ####

#### `:wq` 保存退出 ####

#### 启动服务 ####

```shell
sudo systemctl daemon-reload
sudo systemctl enable ollama
```

#### 启动 `Ollama` ####

```shell
sudo systemctl start ollama
```

#### 查看 `Ollama` 版本 ####

```shell
ollama -v
```

#### 查看服务状态 ####

```shell
sudo systemctl status ollama
```

####  查看日志 ####

```shell
journalctl -e -u ollama
```

## 三、运维管理 ###

### 1、设置环境变量 ### 

#### 编辑配置文件 ####

```ymal
# Ollama 配置文件示例
server:
  host: "0.0.0.0" # 监听所有网络接口
  port: 8080 # 服务端口
  log_level: "info" # 日志级别（debug, info, warn, error）

storage:
  models: "/home/your-username/.ollama/models" # 模型存储路径
  cache: "/home/your-username/.ollama/cache" # 缓存路径

auth:
  enabled: false # 是否启用身份验证
  token: "your-secret-token" # 身份验证令牌
```

#### 编辑 `systemd` 服务 #### 

```shell
systemctl edit ollama.service 
```

#### 在 `[Service]` 部分下添加一行 `Environment` 如下所示 ####

```
[Service]
Environment="OLLAMA_HOST=0.0.0.0"
```

#### 重新加载 `systemd` ####

```shell
systemctl daemon-reload
```

#### 重启服务 ####

```shell
systemctl restart ollama
```

### 2、维护 `Ollama`  模型 ###

> xxx 是指模型名称，具体参考 [模型文件](https://ollama.com/search/)

#### 列出模型列表 ####

```shell
ollama list
```

#### 列出当前已加载的模型 ####

```shell
ollama ps
```

#### 显示模型信息 ####

```shell
ollama show xxx
```

#### 运行模型 ####

```shell
ollama run xxx
```

#### 停止当前正在运行的模型 ####

```shell
ollama stop xxx
```

#### 拉取模型 ####

```shell
ollama pull xxx
```

#### 删除模型 ####

```shell
ollama rm xxx
```

#### 复制模型 ####

```shell
ollama cp xxx model3
```

> 待完善

### 3、更新升级 `Ollama`  ###

#### 更新前先停止服务，再根据自身情况选择AB方案 ####

```shell
sudo systemctl stop ollama
```

#### A、自动更新 ####

> 再次运行安装脚本来更新 `Ollama`

```shell
curl -fsSL https://ollama.com/install.sh | sh
```

#### B、手动更新 ####

> 手动下载

```shell
sudo curl -L https://ollama.com/download/ollama-linux-amd64 -o /usr/bin/ollama
sudo chmod +x /usr/bin/ollama
```

#### 更新完成后尝试启动服务 ####

> 重载服务

```shell
sudo systemctl daemon-reload
```

> 启动 `Ollama`

```shell
sudo systemctl start ollama
```

> 查看服务状态

```shell
sudo systemctl status ollama
```

> 查看日志

```shell
journalctl -u ollama
```

### 4、卸载删除 `Ollama`  ###

#### 卸载服务 ####

```shell
sudo systemctl stop ollama
sudo systemctl disable ollama
sudo rm /etc/systemd/system/ollama.service
```

#### 删除 `Ollama` 二进制文件 ####

```shell
sudo rm $(which ollama)
```

#### 删除已下载的模型以及 `Ollama` 服务用户和组 ####

```shell
sudo rm -r /usr/share/ollama
sudo userdel ollama
sudo groupdel ollama
```
