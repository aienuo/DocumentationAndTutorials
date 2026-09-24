# `Docker Context` 使用指北 #

> **Context（上下文）是 Docker 客户端用来决定"这条命令发给哪个 Docker 守护进程"的一套连接配置**

## 一、`Context` 存在哪里 ##

**Windows：**

```
C:\Users\<你的用户名>\.docker\contexts\
```

**Linux/Mac：**

```
~/.docker/contexts/
```

内部结构：

```
contexts/
└── meta/
    └── sha256/
        └── <哈希>/
            ├── meta.json      # context 元数据（名字、host、证书路径）
            └── tls/           # 证书副本（如果创建时指定了证书）
```

> **重要**：用 `docker context create ... ca=... cert=... key=...` 创建时，Docker 会把证书 **复制一份**到 context
> 目录里。所以原目录的证书删了也不影响，context 自带副本

## 二、查看所有 `Context` ##

```cmd
docker context ls
```

输出示例：

```
NAME            DESCRIPTION                               DOCKER ENDPOINT
default *       Current DOCKER_HOST based configuration   npipe:////./pipe/docker_engine
```

- `*` 表示 **当前正在使用**的 context

## 三、添加新的 `Context` ##

```cmd
docker context create docker-108 ^
  --docker "host=tcp://100.110.111.108:6732,ca=D:\Program Files\JetBrains\docker\108-docker_MiYao\ca.pem,cert=D:\Program Files\JetBrains\docker\108-docker_MiYao\cert.pem,key=D:\Program Files\JetBrains\docker\108-docker_MiYao\key.pem" ^
  --description "在 100.110.111.108:6732 上安装的 X86 Docker_29.3.0"
```

- 看详情：

```cmd
docker context inspect docker-108
```

会显示完整的 `host`、`ca`、`cert`、`key` 路径

## 四、切换指定 `Context` ##

```cmd
docker context use docker-108     # 切到远程
docker context use default        # 切回本地
```

切换后， **所有 `docker` 命令自动走对应端点**，不用再加 `-H` 或证书参数

**临时用某个 context 执行一条命令**（不切换）：

```cmd
docker --context docker-108 ps
docker --context default ps
```

## 五、修改指定 `Context` ##

Docker **没有直接的 `context update` 命令**，要改就得 **删除重建**：

```cmd
docker context rm docker-108
```

```cmd
docker context create docker-108 ^
  --docker "host=tcp://100.110.111.108:6732,ca=D:\Program Files\JetBrains\docker\108-docker_MiYao\ca.pem,cert=D:\Program Files\JetBrains\docker\108-docker_MiYao\cert.pem,key=D:\Program Files\JetBrains\docker\108-docker_MiYao\key.pem" ^
  --description "在 100.110.111.108:6732 上安装的 X86 Docker_29.3.0"
```

**常见修改场景**：

- 服务器 IP 变了
- 证书路径变了
- 端口变了

## 六、删除指定 `Context` ##

```cmd
docker context rm docker-108
```

**注意：**

- 不能删除 **当前正在使用**的 context，会报错：
  ```
  Cannot remove the context currently in use
  ```
  先 `docker context use default`，再执行删除

- 删除 context **不会删除远程主机上的任何东西**，只删本地配置

## 七、`Context` 常见问题 ##

| 问题                           | 原因                   | 解决                                  |
|--------------------------------|------------------------|---------------------------------------|
| `Cannot remove context in use` | 正在用这个 context     | 先 `use default`                      |
| `context not found`            | 名字写错               | `docker context ls` 确认              |
| 切换后连不上                   | IP/端口/证书变了       | 删了重建                              |
| `error during connect`         | 证书路径失效           | `docker context inspect` 看路径，重建 |
| 证书更新了但 context 还用旧的  | context 存的是**副本** | 重建 context                          |
