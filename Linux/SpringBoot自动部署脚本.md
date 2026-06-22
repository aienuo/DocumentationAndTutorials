# SpringBoot Jar包 Linux服务器 开机自启，批量运行


[参考](https://blog.csdn.net/whh18254122507/article/details/78011713)

> 需要注意 `JDK` 安装目录、服务安装位置

## Jar包启动脚本，写个 `service-ctl.sh` 的脚本。注意修改脚本参数 `JAVA_HOME` `JAR_DIR` `LOG_DIR` 

```shell
vim /usr/local/XXX_server/service-ctl.sh
```

##### 按一下键盘字母`i`进行编辑 #####

#### 输入以下内容： ####

```shell
#!/bin/bash

# 启动所有服务
# ./service-ctl.sh start

# 停止所有服务
# ./service-ctl.sh stop

# 重启所有服务
# ./service-ctl.sh restart

# 查看服务状态
# ./service-ctl.sh status

# 基础配置
JAVA_HOME="/usr/local/jdk-17.0.15+6"				# JDK安装目录	
JAR_DIR="/usr/local/shu_ju_zhi_li/jars"				# JAR包存放目录
LOG_DIR="/usr/local/shu_ju_zhi_li/logs"				# 日志输出目录
JVM_OPTS="-Xms512m -Xmx1024m"						# JVM参数配置
SLEEP_TIME=2										# 操作间隔时间（秒）
KILL_TIMEOUT=10										# 停止超时时间（秒）

# 创建日志目录
mkdir -p ${LOG_DIR}

# 启动单个JAR函数
start_jar() {
    local jar_path=$1
    local jar_name=$(basename ${jar_path})
    local log_file="${LOG_DIR}/${jar_name%.*}_$(date "+%Y%m%d_%H%M%S").log"
    local pid_file="${LOG_DIR}/${jar_name%.*}.pid"

    # 检查是否已运行
    if [ -f ${pid_file} ]; then
        local pid=$(cat ${pid_file})
        if ps -p ${pid} > /dev/null; then
            echo "[WARN] ${jar_name} 已在运行中 (PID: ${pid})"
            return 1
        fi
    fi

    # 启动命令
    nohup ${JAVA_HOME}/bin/java ${JVM_OPTS} -jar ${jar_path} > ${log_file} 2>&1 &
    local pid=$!

    # 验证进程是否启动成功
    sleep 1
    if ! ps -p ${pid} > /dev/null; then
        echo "[ERROR] ${jar_name} 启动失败，请检查日志: ${log_file}"
        return 2
    fi

    # 保存PID文件
    echo ${pid} > ${pid_file}
    echo "[INFO] ${jar_name} 启动成功 (PID: ${pid}, 日志: ${log_file})"
}

# 停止单个JAR函数
stop_jar() {
    local pid_file=$1
    local pid=$(cat ${pid_file})
    local jar_name=$(basename ${pid_file%.*})

    if ps -p ${pid} > /dev/null; then
        echo "正在停止 ${jar_name} (PID: ${pid})"
        kill ${pid}

        # 等待进程停止
        local wait_time=0
        while [ ${wait_time} -lt ${KILL_TIMEOUT} ] && ps -p ${pid} > /dev/null; do
            sleep 1
            ((wait_time++))
        done

        if ps -p ${pid} > /dev/null; then
            echo "[WARN] 强制终止 ${jar_name}..."
            kill -9 ${pid}
            sleep 1
        fi

        if ! ps -p ${pid} > /dev/null; then
            echo "[INFO] ${jar_name} 已停止"
            rm -f ${pid_file}
        else
            echo "[ERROR] 无法停止 ${jar_name}"
            return 1
        fi
    else
        echo "[WARN] ${jar_name} 进程未运行"
        rm -f ${pid_file}
    fi
}

# 查询单个JAR状态函数
status_jar() {
    local jar_path=$1
    local jar_name=$(basename ${jar_path})
    local pid_file="${LOG_DIR}/${jar_name%.*}.pid"

    # 检查是否已运行
    if [ -f ${pid_file} ]; then
        local pid=$(cat ${pid_file})
        if ps -p ${pid} > /dev/null; then
			local port=$(sudo netstat -tulnp | grep "${pid}" | awk '{print $4}' | awk -F: '{print $NF}' | sort -n | uniq)
            echo "[INFO] ${jar_name} 已在运行中 (进程PID: ${pid}； 端口Port： ${port})"
		else
			echo "[WARN] ${jar_name} 未运行"
        fi
	else
        echo "[WARN] ${jar_name} 未找到PID文件(${pid_file})"
    fi
}

# 批量启动函数
start_all() {
    local jar_files=($(find ${JAR_DIR} -type f -name "*.jar" | sort))
    if [ ${#jar_files[@]} -eq 0 ]; then
        echo "[ERROR] 未找到JAR文件"
        return 1
    fi

    echo "正在启动 ${#jar_files[@]} 个服务..."
    local count=0
    for jar in "${jar_files[@]}"; do
        ((count++))
        echo "进度: ${count}/${#jar_files[@]}"
        start_jar "${jar}"
        sleep ${SLEEP_TIME}
    done
}

# 批量停止函数
stop_all() {
    local pid_files=($(find ${LOG_DIR} -type f -name "*.pid" | sort -r))
    if [ ${#pid_files[@]} -eq 0 ]; then
        echo "[INFO] 没有运行中的服务"
        return 0
    fi

    echo "正在停止 ${#pid_files[@]} 个服务..."
    local count=0
    for pid_file in "${pid_files[@]}"; do
        ((count++))
        echo "进度: ${count}/${#pid_files[@]}"
        stop_jar "${pid_file}"
        sleep ${SLEEP_TIME}
    done
}

# 批量查看状态函数
status_all() {
    local jar_files=($(find ${JAR_DIR} -type f -name "*.jar" | sort))
    if [ ${#jar_files[@]} -eq 0 ]; then
        echo "[ERROR] 未找到JAR文件"
        return 1
    fi

    echo "正在启动 ${#jar_files[@]} 个服务..."
    local count=0
    for jar in "${jar_files[@]}"; do
        ((count++))
        echo "进度: ${count}/${#jar_files[@]}"
        status_jar "${jar}"
        sleep ${SLEEP_TIME}
    done
}

# 重启函数
restart_all() {
    stop_all
    sleep 2
    start_all
}

# 主程序逻辑
case "$1" in
    start)
        start_all
        ;;
    stop)
        stop_all
        ;;
	status)
        status_all
        ;;
    restart)
        restart_all
        ;;
    *)
        echo "用法: $0 {start|stop|status|restart}"
        exit 1
        ;;
esac

echo "操作执行完成"
```

##### 按一下`esc`键 退出编辑 #####

##### `:wq` 保存退出 #####

## 开机自启脚本

### 创建配置文件 配置开机自启 ###

```shell
vim /lib/systemd/system/XXX.service
```

##### 按一下键盘字母`i`进行编辑 #####

#### 输入以下内容： ####

```shell
[Unit]
Description=XXX_Server
After=network.target

[Service]
# 添加java的环境变量，在systemctl中它不会读取.bash_profile中的环境变量的，必须明确指定
Environment="JAVA_HOME=/usr/local/jdk1.8.0_192"
Type=forking
ExecStart=/usr/local/XXX_server/service-ctl.sh start
ExecReload=/usr/local/XXX_server/service-ctl.sh restart
ExecStop=/usr/local/XXX_server/service-ctl.sh stop
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

##### 按一下`esc`键 退出编辑 #####

##### `:wq` 保存退出 #####

#### 设置开机自启

##### 先进行文件生效配置 #####

```shell
systemctl daemon-reload
```
##### 设置为开机启动 #####

```shell
systemctl enable XXX_Server.service
```

##### 启动服务 #####

```shell
systemctl start XXX_Server.service
```

##### 启动成功后进行查看 #####

```shell
systemctl status XXX_Server.service
```

##### 关闭服务 #####

```shell
systemctl stop XXX_Server.service
```

## 防火墙配置

### 查看防火墙状态:

```text
sudo systemctl status firewalld 
```

### 开启firewall:
```text
service firewalld start
```

### 停止firewall:
```text
systemctl stop firewalld.service
```

### 查询端口是否开放:
```text
firewall-cmd --query-port=8080/tcp
```

### 开放8080端口:
```text
firewall-cmd --permanent --add-port=8080/tcp
```

### 移除端口:
```text
firewall-cmd --permanent --remove-port=8080/tcp
```

### 重启防火墙(修改配置后要重启防火墙!!!):
```text
firewall-cmd --reload
```
