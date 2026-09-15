# Centos10 环境下 Jenkins 安装配置 #

### 注意看我的标题！！！！我这是针对 `2.555.3` 版本的离线（内网）部署 ###

## 一、安装 JDK ##

### 1、下载 ###

[OpenJDK21下载地址](https://mirrors.tuna.tsinghua.edu.cn/Adoptium/21/jdk/x64/linux/OpenJDK21U-jdk_x64_linux_hotspot_21.0.11_10.tar.gz)

### 2、将下载后的文件上传到 `/usr/local` 文件夹下 ###

* 使用 [windows Terminal 工具](https://apps.microsoft.com/detail/9n8g5rfz9xk3?hl=zh-CN&gl=CN)。 语法： scp 【本地文件地址】 【服务器账号】@【服务器IP地址】:/usr/local/

```shell
scp D:/Downloads/OpenJDK21U-jdk_x64_linux_hotspot_21.0.11_10.tar.gz root@100.110.111.114:/usr/local/
```

### 3、远程登录服务器 ###

* 使用 [windows Terminal 工具](https://apps.microsoft.com/detail/9n8g5rfz9xk3?hl=zh-CN&gl=CN)。 语法： ssh 【服务器账号】@【服务器IP地址】

```shell
ssh root@100.110.111.114
```

### 4、进入目录解压 `tar.gz` ###

```shell
cd /usr/local/
```

```shell
tar -zxvf OpenJDK21U-jdk_x64_linux_hotspot_21.0.11_10.tar.gz  -C /usr/local/
```

### 5、配置环境变量 ###

#### 修改配置文件 ####

```shell
echo 'export JENKINS_HOME=/usr/local/jenkins' >> /etc/profile
echo 'export JAVA_HOME=/usr/local/jdk-21.0.11+10' >> /etc/profile
echo 'export PATH=$JAVA_HOME/bin:$PATH' >> /etc/profile
```

#### 修改立即生效 ####

```shell
source /etc/profile
```

#### 验证安装 ####

```shell

java -version
```

## 二、安装 Jenkins （`war` 包安装） ##

*  [稳定版(WAR)列表](https://get.jenkins.io/war-stable/)

### 1、下载 ###

[最新稳定版 `Jenkins.war` 下载地址](https://get.jenkins.io/war-stable/latest/jenkins.war)

### 2、将下载后的文件上传到 `/usr/local` 文件夹下 ###

* 使用 [windows Terminal 工具](https://apps.microsoft.com/detail/9n8g5rfz9xk3?hl=zh-CN&gl=CN)。 语法： scp 【本地文件地址】 【服务器账号】@【服务器IP地址】:/usr/local/

```shell
scp D:/Downloads/jenkins.war root@100.110.111.114:/usr/local/
```

### 3、远程登录服务器 ###

* 使用 [windows Terminal 工具](https://apps.microsoft.com/detail/9n8g5rfz9xk3?hl=zh-CN&gl=CN)。 语法： ssh 【服务器账号】@【服务器IP地址】

```shell
ssh root@100.110.111.114
```

### 4、创建文件夹 ###

```shell
mkdir -p /usr/local/jenkins
```

### 5、设置执行权限 ###

```shell
chmod 755 /usr/local/jenkins.war
```

### 6、移动到 `jenkins` 目录 ###

```shell
mv /usr/local/jenkins.war /usr/local/jenkins/
```

### 7、创建 `jenkins` 运行脚本 ###

#### 创建文件 ####

```shell
touch /usr/local/jenkins/jenkins.sh
```

#### 编辑文件 ####

```shell
vim /usr/local/jenkins/jenkins.sh
```

##### 按一下键盘字母`i`进行编辑 #####

#### 输入以下内容： ####

```shell
#!/bin/bash

# 启动所有服务
# ./jenkins.sh start

# 停止所有服务
# ./jenkins.sh stop

# 重启所有服务
# ./jenkins.sh restart

# 查看服务状态
# ./jenkins.sh status

# 基础配置
JAVA_HOME="/usr/local/jdk-21.0.11+10"				# JDK安装目录	
WAR_PATH="/usr/local/jenkins/jenkins.war"			# WAR包存放位置
JENKINS_HOME="/usr/local/jenkins"					# 指定 JENKINS_HOME
WAR_PORT=80											# WAR端口
LOG_DIR="/usr/local/jenkins/logs"					# 日志输出目录
JVM_OPTS="-Xms512m -Xmx1024m"						# JVM参数配置
SLEEP_TIME=2										# 操作间隔时间（秒）
KILL_TIMEOUT=10										# 停止超时时间（秒）

# 创建日志目录
mkdir -p ${LOG_DIR}

# 启动 WAR 函数
start_war() {
    local war_name=$(basename ${WAR_PATH})
    local log_file="${LOG_DIR}/${war_name%.*}_$(date "+%Y%m%d_%H%M%S").log"
    local pid_file="${LOG_DIR}/${war_name%.*}.pid"

    # 检查是否已运行
    if [ -f ${pid_file} ]; then
        local pid=$(cat ${pid_file})
        if ps -p ${pid} > /dev/null; then
            echo "[WARN] ${war_name} 已在运行中 (PID: ${pid})"
            return 1
        fi
    fi

    # 启动命令
    nohup ${JAVA_HOME}/bin/java -DJENKINS_HOME=${JENKINS_HOME} ${JVM_OPTS} -jar ${WAR_PATH} --httpPort=${WAR_PORT} > ${log_file} 2>&1 &
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

# 停止 WAR 函数
stop_war() {
	local war_name=$(basename ${WAR_PATH})
    local pid_file="${LOG_DIR}/${war_name%.*}.pid"
    local pid=$(cat ${pid_file})

    if ps -p ${pid} > /dev/null; then
        echo "正在停止 ${war_name} (PID: ${pid})"
        kill ${pid}

        # 等待进程停止
        local wait_time=0
        while [ ${wait_time} -lt ${KILL_TIMEOUT} ] && ps -p ${pid} > /dev/null; do
            sleep 1
            ((wait_time++))
        done

        if ps -p ${pid} > /dev/null; then
            echo "[WARN] 强制终止 ${war_name}..."
            kill -9 ${pid}
            sleep 1
        fi

        if ! ps -p ${pid} > /dev/null; then
            echo "[INFO] ${war_name} 已停止"
            rm -f ${pid_file}
        else
            echo "[ERROR] 无法停止 ${war_name}"
            return 1
        fi
    else
        echo "[WARN] ${war_name} 进程未运行"
        rm -f ${pid_file}
    fi
}

# 查询 WAR 状态 函数
status_war() {
    local war_name=$(basename ${WAR_PATH})
    local pid_file="${LOG_DIR}/${war_name%.*}.pid"

    # 检查是否已运行
    if [ -f ${pid_file} ]; then
        local pid=$(cat ${pid_file})
        if ps -p ${pid} > /dev/null; then
			local port=$(sudo netstat -tulnp | grep "${pid}" | awk '{print $4}' | awk -F: '{print $NF}' | sort -n | uniq)
            echo "[INFO] ${war_name} 已在运行中 (进程PID: ${pid}； 端口Port： ${port})"
		else
			echo "[WARN] ${war_name} 未运行"
        fi
	else
        echo "[WARN] ${war_name} 未找到PID文件(${pid_file})"
    fi
}

# 重启 WAR 函数
restart_war() {
    stop_war
    sleep ${SLEEP_TIME}
    start_war
}

# 主程序逻辑
case "$1" in
    start)
        start_war
        ;;
    stop)
        stop_war
        ;;
	status)
        status_war
        ;;
    restart)
        restart_war
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

#### 赋权 ####

```shell
chmod 777 /usr/local/jenkins/jenkins.sh
```

## 三、运行 Jenkins ##

### 1、 切换目录 ###

```shell
cd /usr/local/jenkins
```

### 2、 `jenkins` 启动 ###

```shell
./jenkins.sh start
```

### 3、查询密码并记录下来 ###

```shell
cat /usr/local/jenkins/secrets/initialAdminPassword
```

### 4、防火墙端口开放 ###

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
firewall-cmd --permanent --zone=public --add-port=80/tcp
```

##### Reload 重新加载 #####

```shell
firewall-cmd --reload
```

##### 检查是否生效 ######

```shell
firewall-cmd --zone=public --query-port=80/tcp
```

### 5、浏览器访问，依据提示完成初始化 ###

> http://`ip`:`port`


## 四、配置 `systemd` 服务实现开机自启 ##

### 1、创建配置文件 ###

```shell
vim /lib/systemd/system/jenkins.service
```

##### 按一下键盘字母`i`进行编辑 #####

#### 输入以下内容： ####

```shell
[Unit]
Description=Jenkins_Server
After=network.target

[Service]
# 添加java的环境变量，在systemctl中它不会读取.bash_profile中的环境变量的，必须明确指定
Environment="JAVA_HOME=/usr/local/jdk-21.0.11+10"
Type=forking
ExecStart=/usr/local/jenkins/jenkins.sh start
ExecReload=/usr/local/jenkins/jenkins.sh restart
ExecStop=/usr/local/jenkins/jenkins.sh stop
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

##### 按一下`esc`键 退出编辑 #####

##### `:wq` 保存退出 #####

### 2、设置开机自启 ###

#### 先进行文件生效配置 ####

```shell
systemctl daemon-reload
```
#### 设置为开机启动 ####

```shell
systemctl enable jenkins.service
```

#### 启动服务 ####

```shell
systemctl start jenkins.service
```

#### 启动成功后进行查看 ####

```shell
systemctl status jenkins.service
```

#### 关闭服务 ####

```shell
systemctl stop jenkins.service
```

### 3、重启计算机 ###

```shell
shutdown -r now
```

## 五、离线安装插件（使用 `Plugin Installation Manager Tool`） ##

> 此操作在你的 `Windows` 终端上进行

### 1、下载 `Plugin Installation Manager Tool` ###

> 需要将其跟前面下载的 `jenkins.war` 放在同级目录下

[Plugin Installation Manager Tool](https://github.com/jenkinsci/plugin-installation-manager-tool/releases/latest)

### 2、准备插件列表：创建一个 `plugins.txt` 文件 ###

[Jenkins 官方插件网站](https://plugins.jenkins.io/)

> 需要将其跟前面下载的 `jenkins.war` 和 `jenkins-plugin-manager-x.xx.x.jar` 放在同级目录下

#### 示例 ####

> 每行 <pluginId>:<version> 或仅 <pluginId>，不带版本号以获取最新版本(可能会偶发依赖冲突)

```text
# 基础安全与凭证管理
credentials-binding

# 流水线核心引擎（包含声明式、脚本式Pipeline及Stage View）
workflow-aggregator

# Git 版本控制集成（含Git客户端依赖）
git
git-client
git-parameter

# 任务文件夹管理（用于分类组织Job）
cloudbees-folder

# 构建超时控制（防止任务卡死）
build-timeout

# 构建前后自动清理工作空间（避免旧文件污染）
ws-cleanup

# 控制台输出添加时间戳（便于排查耗时点）
timestamper

# 简体中文本地化（提升团队使用体验）
localization-zh-cn

# Maven 构建支持 (Pipeline Maven Integration)
maven-plugin
pipeline-maven-api
pipeline-maven-database

# Node.js 构建支持
nodejs
```

* 在生产环境中，建议将 `latest` 替换为明确的版本号
* 在生产环境中，建议将 `latest` 替换为明确的版本号

### 3、创建一个 `download-plugins.bat` 插件离线下载脚本 ###

> 需要将其跟前面下载的 `jenkins.war` 、 `jenkins-plugin-manager-x.xx.x.jar` 和 `plugins.txt` 放在同级目录下，准备完成后双击运行即可

```shell
@echo off
setlocal enabledelayedexpansion
chcp 65001 >nul 2>&1

:: ============================================================
:: Jenkins 插件离线下载脚本 (Windows 批处理版)
:: 使用方法：双击运行，或在 cmd 中执行本文件
:: 前置条件：已安装 Java 并加入 PATH
:: ============================================================

:: ---- 用户配置区（请根据实际情况修改） ----
set JENKINS_WAR=.\jenkins.war
set PLUGIN_FILE=.\plugins.txt
set OUTPUT_DIR=.\offline-plugins
:: 请将下面的 jar 文件名修改为实际下载的 Plugin Installation Manager Tool 文件名
set TOOL_JAR=.\jenkins-plugin-manager-x.xx.x.jar
set UPDATE_CENTER=https://updates.jenkins.io/update-center.json
:: -------------------------------------------

:: 检查 Java 是否存在
where java >nul 2>&1
if errorlevel 1 (
    echo [错误] 未找到 Java，请先安装 JDK/JRE 并将 java 加入 PATH。
    pause
    exit /b 1
)

:: 检查必要文件
if not exist "%JENKINS_WAR%" (
    echo [错误] 未找到 Jenkins WAR 文件: %JENKINS_WAR%
    pause
    exit /b 1
)
if not exist "%PLUGIN_FILE%" (
    echo [错误] 未找到插件列表文件: %PLUGIN_FILE%
    pause
    exit /b 1
)
if not exist "%TOOL_JAR%" (
    echo [错误] 未找到工具 JAR 文件: %TOOL_JAR%
    echo 请将下载的 jenkins-plugin-manager-*.jar 重命名为 jenkins-plugin-manager-2.14.0.jar 并放在当前目录，或修改脚本中的 TOOL_JAR 变量。
    pause
    exit /b 1
)

echo [信息] 开始下载插件（使用镜像源: %UPDATE_CENTER%）...
echo.

:: 执行下载
java -jar "%TOOL_JAR%" ^
    --war "%JENKINS_WAR%" ^
    --plugin-file "%PLUGIN_FILE%" ^
    --plugin-download-directory "%OUTPUT_DIR%" ^
    --clean-download-directory ^
    --jenkins-update-center "%UPDATE_CENTER%" ^
    --verbose

if errorlevel 1 (
    echo.
    echo [错误] 插件下载失败，请检查上方日志。
    pause
    exit /b 1
)

:: 统计下载的 .hpi 文件数量
set count_hpi=0
for /f %%f in ('dir /b "%OUTPUT_DIR%\*.hpi" 2^>nul') do set /a count_hpi+=1

:: 统计下载的 .jpi 文件数量
set count_jpi=0
for /f %%f in ('dir /b "%OUTPUT_DIR%\*.jpi" 2^>nul') do set /a count_jpi+=1

echo.
echo [成功] 插件下载完成，共 %count_hpi% 个 .hpi 文件，%count_jpi% 个 .jpi 文件
echo [提示] 文件存放目录: %OUTPUT_DIR%
echo [提示] 请将此目录内容复制到 Linux 服务器的 JENKINS_HOME/plugins 目录下，然后重启 Jenkins。
pause
```

### 4、将插件文件夹 `offline-plugins` 压缩并上传到服务器 ###

* 使用 [windows Terminal 工具](https://apps.microsoft.com/detail/9n8g5rfz9xk3?hl=zh-CN&gl=CN)。 语法： scp 【本地文件地址】 【服务器账号】@【服务器IP地址】:/usr/local

```shell
scp D:/Downloads/jenkins/offline-plugins.zip root@100.110.111.114:/usr/local/jenkins/
```

### 5、远程登录服务器 ###

* 使用 [windows Terminal 工具](https://apps.microsoft.com/detail/9n8g5rfz9xk3?hl=zh-CN&gl=CN)。 语法： ssh 【服务器账号】@【服务器IP地址】

```shell
ssh root@100.110.111.114
```

### 6、进入目录，将 `offline-plugins.zip` 解压 ###

```shell
cd /usr/local/jenkins/
```

```shell
unzip offline-plugins.zip
```

### 7、将文件夹内的插件文件 移动到 `plugins` 目录下 ###

```shell
mv /usr/local/jenkins/offline-plugins/*.{hpi,jpi} $JENKINS_HOME/plugins/
```

* 删除文件：

```shell
rm -rf /usr/local/jenkins/offline-plugins
```

* 切换到插件文件夹：

```shell
cd /usr/local/jenkins/plugins/
```

* 设置文件权限：

```shell
chmod 755 *.jpi
chmod 755 *.hpi
```

### 8、重启 `Jenkins` 服务 ###

```shell
systemctl restart jenkins.service
```

### 9、启动成功后进行查看 ###

```shell
systemctl status jenkins.service
```

## 六、`Jenkins` 插件批量备份 ##

> 在 `Jenkins` 脚本命令行中执行以下 Groovy 脚本，导出当前已安装插件列表：

```groovy
def plugins = Jenkins.instance.getPluginManager().getPlugins()
plugins.each { plugin ->
    println("${plugin.shortName}:${plugin.version}")
}
```

* 将输出保存为 `plugins.txt`，用于后续恢复或迁移。
