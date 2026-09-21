# Docker 容器排障与运维命令速查表

下面按 **使用场景**分类整理，方便实际排障时快速查阅。

---

# 一、容器生命周期

| 命令                                                    | 作用                            |
|---------------------------------------------------------|---------------------------------|
| `docker run -d --name x nginx`                          | 创建并后台启动容器              |
| `docker start x`                                        | 启动已停止容器                  |
| `docker stop x`                                         | 优雅停止（SIGTERM→10s→SIGKILL） |
| `docker restart x`                                      | 重启（stop+start）              |
| `docker kill x`                                         | 立即强杀                        |
| `docker pause x` / `unpause x`                          | 暂停/恢复                       |
| `docker rm x`                                           | 删除已停止容器                  |
| `docker rm -f x`                                        | 强制删除运行中容器              |
| `docker ps`                                             | 查看运行中容器                  |
| `docker ps -a`                                          | 查看所有容器                    |
| `docker ps -a --format "table {{.Names}}\t{{.Status}}"` | 自定义列                        |

**批量清理：**

```bash
docker stop $(docker ps -q)              # 停所有
docker rm $(docker ps -aq)               # 删所有已停
docker container prune                   # 删所有已停容器
```

---

# 二、日志与输出

| 命令                                          | 作用        |
|-----------------------------------------------|-------------|
| `docker logs x`                               | 全部日志    |
| `docker logs -f x`                            | 实时跟踪    |
| `docker logs --tail 100 x`                    | 最后 100 行 |
| `docker logs -f --tail 50 -t x`               | 跟踪+时间戳 |
| `docker logs --since 10m x`                   | 近 10 分钟  |
| `docker logs --since "2026-09-20T10:00:00" x` | 指定时间起  |
| `docker logs x > x.log 2>&1`                  | 导出到文件  |

**日志占磁盘排查：**

```bash
du -sh /var/lib/docker/containers/*/*-json.log | sort -h | tail
truncate -s 0 /var/lib/docker/containers/<id>/<id>-json.log   # 清空
```

**全局限制日志大小（/etc/docker/daemon.json）：**

```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
```

---

# 三、进入容器 / 执行命令

| 命令                                     | 作用          |
|------------------------------------------|---------------|
| `docker exec -it x bash`                 | 进入 bash     |
| `docker exec -it x sh`                   | 精简镜像用 sh |
| `docker exec -it -u root x bash`         | 以 root 进入  |
| `docker exec -it -w /app x bash`         | 指定工作目录  |
| `docker exec x ps aux`                   | 容器内进程    |
| `docker exec x env`                      | 环境变量      |
| `docker exec x cat /etc/os-release`      | 看系统版本    |
| `docker exec -it redis7 redis-cli`       | 进 Redis CLI  |
| `docker exec -it mysql8 mysql -uroot -p` | 进 MySQL      |

**exec vs attach：**

- `exec`：新开进程，`exit` **不影响容器** ✅
- `attach`：接入主进程，`Ctrl+C` **可能停容器** ⚠️

---

# 四、文件拷贝

| 命令                                   | 作用             |
|----------------------------------------|------------------|
| `docker cp x:/path/file ./`            | 容器 → 宿主机    |
| `docker cp ./file x:/path/`            | 宿主机 → 容器    |
| `docker cp x:/etc/nginx/nginx.conf ./` | 拷配置文件出来改 |
| `docker cp ./nginx.conf x:/etc/nginx/` | 改完拷回去       |

> 注意：`docker cp` 可用于 **已停止**的容器，`exec` 不行。

---

# 五、容器详情与监控

| 命令                                                                   | 作用                     |
|------------------------------------------------------------------------|--------------------------|
| `docker inspect x`                                                     | 全部详情 JSON            |
| `docker inspect --format '{{.State.Status}}' x`                        | 运行状态                 |
| `docker inspect --format '{{.State.Pid}}' x`                           | 主进程 PID               |
| `docker inspect --format '{{.NetworkSettings.IPAddress}}' x`           | 容器 IP                  |
| `docker inspect --format '{{json .Mounts}}' x \| python3 -m json.tool` | 挂载信息                 |
| `docker inspect --format '{{.HostConfig.RestartPolicy.Name}}' x`       | 重启策略                 |
| `docker stats`                                                         | 实时资源占用             |
| `docker stats --no-stream`                                             | 一次性快照               |
| `docker top x`                                                         | 容器内进程（宿主机视角） |
| `docker port x`                                                        | 端口映射                 |
| `docker diff x`                                                        | 容器文件系统改动         |
| `docker events`                                                        | 实时事件流               |

**常用 inspect 格式化：**

```bash
# 看容器 IP
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' x

# 看所有容器的名字+状态
docker inspect -f '{{.Name}} {{.State.Status}}' $(docker ps -aq)

# 看健康检查状态
docker inspect -f '{{.State.Health.Status}}' x
```

---

# 六、镜像相关

| 命令                              | 作用                        |
|-----------------------------------|-----------------------------|
| `docker images`                   | 列出镜像                    |
| `docker pull nginx:1.25`          | 拉取                        |
| `docker rmi nginx:1.25`           | 删除镜像                    |
| `docker tag <id> nginx:1.25`      | 打标签                      |
| `docker history nginx:1.25`       | 看镜像层历史                |
| `docker inspect nginx:1.25`       | 镜像详情（架构/OS/CMD/ENV） |
| `docker image prune`              | 清理悬空镜像                |
| `docker image prune -a`           | 清理所有未使用镜像          |
| `docker save -o x.tar nginx:1.25` | 导出镜像                    |
| `docker load -i x.tar`            | 导入镜像                    |

**查看架构：**

```bash
docker inspect -f '{{.Architecture}}/{{.Os}}' nginx:1.25
```

---

# 七、构建与提交

| 命令                                            | 作用                   |
|-------------------------------------------------|------------------------|
| `docker build -t myapp:v1 .`                    | 构建镜像               |
| `docker build -t myapp:v1 -f Dockerfile.prod .` | 指定 Dockerfile        |
| `docker build --no-cache -t myapp:v1 .`         | 不用缓存               |
| `docker commit x myimg:v1`                      | 容器存成镜像（带配置） |
| `docker export -o x.tar x`                      | 容器导出 rootfs        |
| `docker import x.tar myimg:v1`                  | rootfs 导入成镜像      |

**save/export 区别速记：**

- `save`/`load` → 操作 **镜像**，保留层和历史
- `export`/`import` → 操作 **容器**，压成单层，丢元数据
- `commit` → 容器变镜像（推荐替代 export）

---

# 八、网络

| 命令                                | 作用           |
|-------------------------------------|----------------|
| `docker network ls`                 | 列出网络       |
| `docker network create mynet`       | 创建网络       |
| `docker network inspect mynet`      | 网络详情       |
| `docker network connect mynet x`    | 容器接入网络   |
| `docker network disconnect mynet x` | 断开           |
| `docker network prune`              | 清理未使用网络 |

**容器互联：** 同一自定义网络内，可直接用 **容器名**当主机名访问。

```bash
docker network create appnet
docker run -d --name db --network appnet mysql:8
docker run -d --name web --network appnet nginx
docker exec web ping db    # 可直接用名字
```

---

# 九、数据卷

| 命令                          | 作用                 |
|-------------------------------|----------------------|
| `docker volume ls`            | 列出卷               |
| `docker volume create myvol`  | 创建卷               |
| `docker volume inspect myvol` | 卷详情（宿主机路径） |
| `docker volume rm myvol`      | 删除卷               |
| `docker volume prune`         | 清理未使用卷         |

**卷 vs 绑定挂载：**

- `-v myvol:/data` → **命名卷**，Docker 管理，推荐
- `-v /host/path:/data` → **绑定挂载**，直接映射宿主机目录

---

# 十、资源限制

| 参数                  | 作用          |
|-----------------------|---------------|
| `--memory 512m`       | 限制内存      |
| `--memory-swap 1g`    | 内存+swap     |
| `--cpus 1.5`          | 限制 CPU 核数 |
| `--cpuset-cpus "0,1"` | 绑定 CPU      |
| `--pids-limit 100`    | 限制进程数    |
| `--blkio-weight 500`  | 块设备权重    |

```bash
docker run -d --name x --memory 512m --cpus 1 nginx:1.25
```

---

# 十一、重启策略

| 策略             | 含义                     |
|------------------|--------------------------|
| `no`             | 不自动重启（默认）       |
| `on-failure[:n]` | 非0退出时重启，最多 n 次 |
| `always`         | 总是重启（生产常用）     |
| `unless-stopped` | 除非手动停，否则重启     |

```bash
docker run -d --restart always nginx:1.25
docker update --restart always x    # 修改已运行容器
```

---

# 十二、系统清理（释放磁盘）

```bash
docker system df                    # 查看占用
docker system prune                 # 清理：停容器+悬空镜像+未用网络
docker system prune -a              # 更彻底（含未用镜像）
docker system prune -a --volumes    # 连卷一起清（慎用！）
```

**分项清理：**

```bash
docker container prune    # 已停容器
docker image prune         # 悬空镜像
docker image prune -a      # 所有未用镜像
docker volume prune        # 未用卷
docker network prune       # 未用网络
docker builder prune       # 构建缓存
```

---

# 十三、排障万能六连

容器出问题时，按这个顺序排查：

```bash
# 1. 容器在不在、状态如何
docker ps -a

# 2. 看日志（最直接）
docker logs --tail 200 x

# 3. 看详情（退出码/挂载/网络/重启策略）
docker inspect x

# 4. 看资源占用
docker stats --no-stream x

# 5. 进容器看进程和文件
docker exec -it x sh
#    → ps aux / ls / cat 配置 / netstat

# 6. 看文件改动
docker diff x
```

**关键：先看 `docker logs` 和 `docker inspect` 的 `State`（ExitCode、Error、OOMKilled）。**

---

# 十四、常见故障 → 命令对照

| 故障现象       | 首选命令                                                     |
|----------------|--------------------------------------------------------------|
| 容器启动即退出 | `docker logs x`、`docker inspect -f '{{.State.ExitCode}}' x` |
| 端口访问不通   | `docker port x`、`docker inspect` 网络、`ss -lntp`           |
| 磁盘满         | `docker system df`、`du -sh /var/lib/docker/*`               |
| 内存被打爆     | `docker stats`、`inspect` 看 `OOMKilled`                     |
| 容器间不通     | `docker network inspect`、`exec ping`                        |
| 配置改了不生效 | `docker cp` 出来看、`docker exec cat`                        |
| 镜像架构不对   | `docker inspect -f '{{.Architecture}}'`                      |
| 权限问题       | `docker exec -u root`、`ls -l` 挂载目录                      |

---

# 十五、一页纸速记

```
【看】ps -a / logs -f / inspect / stats / top / port / diff
【进】exec -it x bash（安全，exit 不停容器）
【拷】cp x:/path ./（停止容器也能用）
【停】stop（优雅）/ kill（强杀）
【删】rm / rmi / system prune
【查】网络 network / 卷 volume / 镜像 image
【诊】先 logs 再 inspect，看 ExitCode/OOMKilled
```

---
